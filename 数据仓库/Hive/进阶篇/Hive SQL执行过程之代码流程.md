hive执行sql有两种方式：

- 执行hive命令，又细分为hive -e，hive -f，hive交互式；
- 执行beeline命令，beeline会连接远程thrift server；

下面分别看这些场景下sql是怎样被执行的：

# 1 hive命令

## 启动命令

启动hive客户端命令

> $HIVE_HOME/bin/hive

等价于

> $HIVE_HOME/bin/hive --service cli

会调用

> $HIVE_HOME/bin/ext/cli.sh

实际启动类为：org.apache.hadoop.hive.cli.CliDriver

## 代码解析

org.apache.hadoop.hive.cli.CliDriver

```
  public static void main(String[] args) throws Exception {
    int ret = new CliDriver().run(args);
    System.exit(ret);
  }   public  int run(String[] args) throws Exception {
...
    // execute cli driver work
    try {
      return executeDriver(ss, conf, oproc);
    } finally {
      ss.resetThreadName();
      ss.close();
    }
...   private int executeDriver(CliSessionState ss, HiveConf conf, OptionsProcessor oproc)
      throws Exception {
...
    if (ss.execString != null) {
      int cmdProcessStatus = cli.processLine(ss.execString);
      return cmdProcessStatus;
    }
...
    try {
      if (ss.fileName != null) {
        return cli.processFile(ss.fileName);
      }
    } catch (FileNotFoundException e) {
      System.err.println("Could not open input file for reading. (" + e.getMessage() + ")");
      return 3;
    }
...
    while ((line = reader.readLine(curPrompt + "> ")) != null) {
      if (!prefix.equals("")) {
        prefix += '\n';
      }
      if (line.trim().startsWith("--")) {
        continue;
      }
      if (line.trim().endsWith(";") && !line.trim().endsWith("\\;")) {
        line = prefix + line;
        ret = cli.processLine(line, true);
...   public int processFile(String fileName) throws IOException {
...
      rc = processReader(bufferReader);
...   public int processReader(BufferedReader r) throws IOException {
    String line;
    StringBuilder qsb = new StringBuilder();     while ((line = r.readLine()) != null) {
      // Skipping through comments
      if (! line.startsWith("--")) {
        qsb.append(line + "\n");
      }
    }     return (processLine(qsb.toString()));
  }   public int processLine(String line, boolean allowInterrupting) {
...
        ret = processCmd(command);
...   public int processCmd(String cmd) {
...
        CommandProcessor proc = CommandProcessorFactory.get(tokens, (HiveConf) conf);
        ret = processLocalCmd(cmd, proc, ss);
...   int processLocalCmd(String cmd, CommandProcessor proc, CliSessionState ss) {
    int tryCount = 0;
    boolean needRetry;
    int ret = 0;     do {
      try {
        needRetry = false;
        if (proc != null) {
          if (proc instanceof Driver) {
            Driver qp = (Driver) proc;
            PrintStream out = ss.out;
            long start = System.currentTimeMillis();
            if (ss.getIsVerbose()) {
              out.println(cmd);
            }             qp.setTryCount(tryCount);
            ret = qp.run(cmd).getResponseCode();
...
              while (qp.getResults(res)) {
                for (String r : res) {
                  out.println(r);
                }
...
```

CliDriver.main会调用run，run会调用executeDriver，在executeDriver中对应上边提到的三种情况：

- 一种是hive -e执行sql，此时ss.execString非空，执行完进程退出；
- 一种是hive -f执行sql文件，此时ss.fileName非空，执行完进程退出；
- 一种是hive交互式执行sql，此时会不断读取reader.readLine，然后执行失去了并输出结果；

上述三种情况最终都会调用processLine，processLine会调用processLocalCmd，在processLocalCmd中会先调用到Driver.run执行sql，执行完之后再调用Driver.getResults输出结果，这也是Driver最重要的两个接口，Driver实现后边再看；

# 2 beeline命令

beeline需要连接到hive thrift server，先看hive thrift server如何启动：

## hive thrift server

### 启动命令

启动hive thrift server命令

> $HIVE_HOME/bin/hiveserver2

等价于

> $HIVE_HOME/bin/hive --service hiveserver2

会调用

> $HIVE_HOME/bin/ext/hiveserver2.sh

实际启动类为：org.apache.hive.service.server.HiveServer2

### 启动过程

> HiveServer2.main
>
> startHiveServer2
>
> init
>
> addService-CLIService,ThriftBinaryCLIService
>
> start
>
> Service.start
>
> CLIService.start
>
> ThriftBinaryCLIService.start
>
> TThreadPoolServer.serve

### 类结构：【接口或父类->子类】

> TServer->TThreadPoolServer
>
> TProcessorFactory->SQLPlainProcessorFactory
>
> TProcessor->TSetIpAddressProcessor
>
> ThriftCLIService->ThriftBinaryCLIService
>
> CLIService
>
> HiveSession

### 代码解析

org.apache.hive.service.cli.thrift.ThriftBinaryCLIService

```
  public ThriftBinaryCLIService(CLIService cliService, Runnable oomHook) {
    super(cliService, ThriftBinaryCLIService.class.getSimpleName());
    this.oomHook = oomHook;
  }
```

ThriftBinaryCLIService是一个核心类，其中会实际启动thrift server，同时包装一个CLIService，请求最后都会调用底层的CLIService处理，下面看CLIService代码：

org.apache.hive.service.cli.CLIService

```
  @Override
  public OperationHandle executeStatement(SessionHandle sessionHandle, String statement,
      Map<String, String> confOverlay) throws HiveSQLException {
    OperationHandle opHandle =
        sessionManager.getSession(sessionHandle).executeStatement(statement, confOverlay);
    LOG.debug(sessionHandle + ": executeStatement()");
    return opHandle;
  }   @Override
  public RowSet fetchResults(OperationHandle opHandle, FetchOrientation orientation,
                             long maxRows, FetchType fetchType) throws HiveSQLException {
    RowSet rowSet = sessionManager.getOperationManager().getOperation(opHandle)
        .getParentSession().fetchResults(opHandle, orientation, maxRows, fetchType);
    LOG.debug(opHandle + ": fetchResults()");
    return rowSet;
  }
```

CLIService最重要的两个接口，一个是executeStatement，一个是fetchResults，两个接口都会转发给HiveSession处理，下面看HiveSession实现类代码：

org.apache.hive.service.cli.session.HiveSessionImpl

```
  @Override
  public OperationHandle executeStatement(String statement, Map<String, String> confOverlay) throws HiveSQLException {
    return executeStatementInternal(statement, confOverlay, false, 0);
  }   private OperationHandle executeStatementInternal(String statement,
      Map<String, String> confOverlay, boolean runAsync, long queryTimeout) throws HiveSQLException {
    acquire(true, true);     ExecuteStatementOperation operation = null;
    OperationHandle opHandle = null;
    try {
      operation = getOperationManager().newExecuteStatementOperation(getSession(), statement,
          confOverlay, runAsync, queryTimeout);
      opHandle = operation.getHandle();
      operation.run();
...
  @Override
  public RowSet fetchResults(OperationHandle opHandle, FetchOrientation orientation,
      long maxRows, FetchType fetchType) throws HiveSQLException {
    acquire(true, false);
    try {
      if (fetchType == FetchType.QUERY_OUTPUT) {
        return operationManager.getOperationNextRowSet(opHandle, orientation, maxRows);
      }
      return operationManager.getOperationLogRowSet(opHandle, orientation, maxRows, sessionConf);
    } finally {
      release(true, false);
    }
  }
```

可见

- HiveSessionImpl.executeStatement是调用ExecuteStatementOperation.run（ExecuteStatementOperation是Operation的一种）
- HiveSessionImpl.fetchResults是调用OperationManager.getOperationNextRowSet，然后会调用到Operation.getNextRowSet

org.apache.hive.service.cli.operation.OperationManager

```
  public RowSet getOperationNextRowSet(OperationHandle opHandle,
      FetchOrientation orientation, long maxRows)
          throws HiveSQLException {
    return getOperation(opHandle).getNextRowSet(orientation, maxRows);
  }
```

下面写详细看Operation的run和getOperationNextRowSet：

org.apache.hive.service.cli.operation.Operation

```
  public void run() throws HiveSQLException {
    beforeRun();
    try {
      Metrics metrics = MetricsFactory.getInstance();
      if (metrics != null) {
        try {
          metrics.incrementCounter(MetricsConstant.OPEN_OPERATIONS);
        } catch (Exception e) {
          LOG.warn("Error Reporting open operation to Metrics system", e);
        }
      }
      runInternal();
    } finally {
      afterRun();
    }
  }   public RowSet getNextRowSet() throws HiveSQLException {
    return getNextRowSet(FetchOrientation.FETCH_NEXT, DEFAULT_FETCH_MAX_ROWS);
  }
```

Operation是一个抽象类，

- run会调用抽象方法runInternal
- getNextRowSet会调用抽象方法getNextRowSet

下面会看到这两个抽象方法在子类中的实现，最终会依赖Driver的run和getResults；

1）先看runInternal在子类HiveCommandOperation中被实现：

org.apache.hive.service.cli.operation.HiveCommandOperation

```
  @Override
  public void runInternal() throws HiveSQLException {
    setState(OperationState.RUNNING);
    try {
      String command = getStatement().trim();
      String[] tokens = statement.split("\\s");
      String commandArgs = command.substring(tokens[0].length()).trim();       CommandProcessorResponse response = commandProcessor.run(commandArgs);
...
```

这里会调用CommandProcessor.run，实际会调用Driver.run（Driver是CommandProcessor的实现类）；

2）再看getNextRowSet在子类SQLOperation中被实现：

org.apache.hive.service.cli.operation.SQLOperation

```
  public RowSet getNextRowSet(FetchOrientation orientation, long maxRows)
    throws HiveSQLException {
...
      driver.setMaxRows((int) maxRows);
      if (driver.getResults(convey)) {
        return decode(convey, rowSet);
      }
...
```

这里会调用Driver.getResults；

# 3 Driver

通过上面的代码分析发现无论是hive命令行执行还是beeline连接thrift server执行，最终都会依赖Driver，

Driver最核心的两个接口：

- run
- getResults

## 代码解析

org.apache.hadoop.hive.ql.Driver

```
  @Override
  public CommandProcessorResponse run(String command)
      throws CommandNeedRetryException {
    return run(command, false);
  }   public CommandProcessorResponse run(String command, boolean alreadyCompiled)
        throws CommandNeedRetryException {
    CommandProcessorResponse cpr = runInternal(command, alreadyCompiled);
...
  private CommandProcessorResponse runInternal(String command, boolean alreadyCompiled)
      throws CommandNeedRetryException {
...
        ret = compileInternal(command, true);
...
      ret = execute(true);
...
  private int compileInternal(String command, boolean deferClose) {
...
      ret = compile(command, true, deferClose);
...
  public int compile(String command, boolean resetTaskIds, boolean deferClose) {
...
      plan = new QueryPlan(queryStr, sem, perfLogger.getStartTime(PerfLogger.DRIVER_RUN), queryId,
        queryState.getHiveOperation(), schema);
...
  public int execute(boolean deferClose) throws CommandNeedRetryException {
...
      // Add root Tasks to runnable
      for (Task<? extends Serializable> tsk : plan.getRootTasks()) {
        // This should never happen, if it does, it's a bug with the potential to produce
        // incorrect results.
        assert tsk.getParentTasks() == null || tsk.getParentTasks().isEmpty();
        driverCxt.addToRunnable(tsk);
      }
...
      // Loop while you either have tasks running, or tasks queued up
      while (driverCxt.isRunning()) {         // Launch upto maxthreads tasks
        Task<? extends Serializable> task;
        while ((task = driverCxt.getRunnable(maxthreads)) != null) {
          TaskRunner runner = launchTask(task, queryId, noName, jobname, jobs, driverCxt);
          if (!runner.isRunning()) {
            break;
          }
        }         // poll the Tasks to see which one completed
        TaskRunner tskRun = driverCxt.pollFinished();
        if (tskRun == null) {
          continue;
        }
        hookContext.addCompleteTask(tskRun);
        queryDisplay.setTaskResult(tskRun.getTask().getId(), tskRun.getTaskResult());         Task<? extends Serializable> tsk = tskRun.getTask();
        TaskResult result = tskRun.getTaskResult();
...
        if (tsk.getChildTasks() != null) {
          for (Task<? extends Serializable> child : tsk.getChildTasks()) {
            if (DriverContext.isLaunchable(child)) {
              driverCxt.addToRunnable(child);
            }
          }
        }
      }   public boolean getResults(List res) throws IOException, CommandNeedRetryException {
    if (driverState == DriverState.DESTROYED || driverState == DriverState.CLOSED) {
      throw new IOException("FAILED: query has been cancelled, closed, or destroyed.");
    }     if (isFetchingTable()) {
      /**
       * If resultset serialization to thrift object is enabled, and if the destination table is
       * indeed written using ThriftJDBCBinarySerDe, read one row from the output sequence file,
       * since it is a blob of row batches.
       */
      if (fetchTask.getWork().isUsingThriftJDBCBinarySerDe()) {
        maxRows = 1;
      }
      fetchTask.setMaxRows(maxRows);
      return fetchTask.fetch(res);
    }
...
```

- Driver的run会调用runInternal，runInternal中会先compileInternal编译sql并生成QueryPlan，然后调用execute执行QueryPlan中的所有task；
- Driver的getResults会调用FetchTask的fetch来获取结果；