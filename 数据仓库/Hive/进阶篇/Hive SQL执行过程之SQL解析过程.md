# Hive SQL解析过程

> SQL->AST(Abstract Syntax Tree)->Task（MapRedTask，FetchTask）->QueryPlan（Task集合）->Job（Yarn）

SQL解析会在两个地方进行：

- 一个是SQL执行前compile，具体在Driver.compile，为了创建QueryPlan；
- 一个是explain，具体在ExplainSemanticAnalyzer.analyzeInternal，为了创建ExplainTask；

# SQL执行过程

## 1 compile过程（SQL->AST(Abstract Syntax Tree)->QueryPlan）

org.apache.hadoop.hive.ql.Driver

```
  public int compile(String command, boolean resetTaskIds, boolean deferClose) {
...
      ParseDriver pd = new ParseDriver();
      ASTNode tree = pd.parse(command, ctx);
      tree = ParseUtils.findRootNonNullToken(tree);
...
      BaseSemanticAnalyzer sem = SemanticAnalyzerFactory.get(queryState, tree);
...
        sem.analyze(tree, ctx);
...
      // Record any ACID compliant FileSinkOperators we saw so we can add our transaction ID to
      // them later.
      acidSinks = sem.getAcidFileSinks();       LOG.info("Semantic Analysis Completed");       // validate the plan
      sem.validate();
      acidInQuery = sem.hasAcidInQuery();
      perfLogger.PerfLogEnd(CLASS_NAME, PerfLogger.ANALYZE);       if (isInterrupted()) {
        return handleInterruption("after analyzing query.");
      }       // get the output schema
      schema = getSchema(sem, conf);
      plan = new QueryPlan(queryStr, sem, perfLogger.getStartTime(PerfLogger.DRIVER_RUN), queryId,
        queryState.getHiveOperation(), schema);
...
```

compile过程为先由ParseDriver将SQL转换为ASTNode，然后由BaseSemanticAnalyzer对ASTNode进行分析，最后将BaseSemanticAnalyzer传入QueryPlan构造函数来创建QueryPlan；

### 1）将SQL转换为ASTNode过程如下（SQL->AST(Abstract Syntax Tree)）

org.apache.hadoop.hive.ql.parse.ParseDriver

```
  public ASTNode parse(String command, Context ctx, boolean setTokenRewriteStream)
      throws ParseException {
    if (LOG.isDebugEnabled()) {
      LOG.debug("Parsing command: " + command);
    }     HiveLexerX lexer = new HiveLexerX(new ANTLRNoCaseStringStream(command));
    TokenRewriteStream tokens = new TokenRewriteStream(lexer);
    if (ctx != null) {
      if ( setTokenRewriteStream) {
        ctx.setTokenRewriteStream(tokens);
      }
      lexer.setHiveConf(ctx.getConf());
    }
    HiveParser parser = new HiveParser(tokens);
    if (ctx != null) {
      parser.setHiveConf(ctx.getConf());
    }
    parser.setTreeAdaptor(adaptor);
    HiveParser.statement_return r = null;
    try {
      r = parser.statement();
    } catch (RecognitionException e) {
      e.printStackTrace();
      throw new ParseException(parser.errors);
    }     if (lexer.getErrors().size() == 0 && parser.errors.size() == 0) {
      LOG.debug("Parse Completed");
    } else if (lexer.getErrors().size() != 0) {
      throw new ParseException(lexer.getErrors());
    } else {
      throw new ParseException(parser.errors);
    }     ASTNode tree = (ASTNode) r.getTree();
    tree.setUnknownTokenBoundaries();
    return tree;
  }
```

### 2）analyze过程（AST(Abstract Syntax Tree)->Task）

org.apache.hadoop.hive.ql.parse.BaseSemanticAnalyzer

```
  public void analyze(ASTNode ast, Context ctx) throws SemanticException {
    initCtx(ctx);
    init(true);
    analyzeInternal(ast);
  }
```

其中analyzeInternal是抽象方法，由不同的子类实现，比如DDLSemanticAnalyzer，SemanticAnalyzer，UpdateDeleteSemanticAnalyzer，ExplainSemanticAnalyzer等；
analyzeInternal主要的工作是将ASTNode转化为Task，包括可能的optimize，过程比较复杂，这里不贴代码；

### 3）创建QueryPlan过程如下（Task->QueryPlan）

org.apache.hadoop.hive.ql.QueryPlan

```
  public QueryPlan(String queryString, BaseSemanticAnalyzer sem, Long startTime, String queryId,
                  HiveOperation operation, Schema resultSchema) {
    this.queryString = queryString;     rootTasks = new ArrayList<Task<? extends Serializable>>(sem.getAllRootTasks());
    reducerTimeStatsPerJobList = new ArrayList<ReducerTimeStatsPerJob>();
    fetchTask = sem.getFetchTask();
    // Note that inputs and outputs can be changed when the query gets executed
    inputs = sem.getAllInputs();
    outputs = sem.getAllOutputs();
    linfo = sem.getLineageInfo();
    tableAccessInfo = sem.getTableAccessInfo();
    columnAccessInfo = sem.getColumnAccessInfo();
    idToTableNameMap = new HashMap<String, String>(sem.getIdToTableNameMap());     this.queryId = queryId == null ? makeQueryId() : queryId;
    query = new org.apache.hadoop.hive.ql.plan.api.Query();
    query.setQueryId(this.queryId);
    query.putToQueryAttributes("queryString", this.queryString);
    queryProperties = sem.getQueryProperties();
    queryStartTime = startTime;
    this.operation = operation;
    this.autoCommitValue = sem.getAutoCommitValue();
    this.resultSchema = resultSchema;
  }
```

可见只是简单的将BaseSemanticAnalyzer中的内容拷贝出来，其中最重要的是sem.getAllRootTasks和sem.getFetchTask；

## 2 execute过程（QueryPlan->Job）

org.apache.hadoop.hive.ql.Driver

```
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
        }
...   private TaskRunner launchTask(Task<? extends Serializable> tsk, String queryId, boolean noName,
      String jobname, int jobs, DriverContext cxt) throws HiveException {
...
    TaskRunner tskRun = new TaskRunner(tsk, tskRes);
...
      tskRun.start();
...
      tskRun.runSequential();
...
```

Driver.run中从QueryPlan中取出Task，并逐个launchTask，launchTask过程为将Task包装为TaskRunner，并最终调用TaskRunner.runSequential，下面看TaskRunner：

org.apache.hadoop.hive.ql.exec.TaskRunner

```
  public void runSequential() {
    int exitVal = -101;
    try {
      exitVal = tsk.executeTask();
...
```

这里直接调用Task.executeTask

org.apache.hadoop.hive.ql.exec.Task

```
  public int executeTask() {
...
      int retval = execute(driverContext);
...
```

这里execute是抽象方法，由子类实现，比如DDLTask，MapRedTask等，着重看MapRedTask，因为大部分的Task都是MapRedTask：

org.apache.hadoop.hive.ql.exec.mr.MapRedTask

```
  public int execute(DriverContext driverContext) {
...
      if (!runningViaChild) {
        // we are not running this mapred task via child jvm
        // so directly invoke ExecDriver
        return super.execute(driverContext);
      }
...
```

这里直接调用父类方法，也就是ExecDriver.execute，下面看：

org.apache.hadoop.hive.ql.exec.mr.ExecDriver

```
  protected transient JobConf job;
...
  public int execute(DriverContext driverContext) {
...
    JobClient jc = null;     MapWork mWork = work.getMapWork();
    ReduceWork rWork = work.getReduceWork();
...
    if (mWork.getNumMapTasks() != null) {
      job.setNumMapTasks(mWork.getNumMapTasks().intValue());
    }
...
    job.setNumReduceTasks(rWork != null ? rWork.getNumReduceTasks().intValue() : 0);
    job.setReducerClass(ExecReducer.class);
...
      jc = new JobClient(job);
...
      rj = jc.submitJob(job);
      this.jobID = rj.getJobID();
...
```

这里将Task转化为Job提交到Yarn执行；

# SQL Explain过程

另外一个SQL解析的过程是explain，在ExplainSemanticAnalyzer中将ASTNode转化为ExplainTask：

org.apache.hadoop.hive.ql.parse.ExplainSemanticAnalyzer

```
  public void analyzeInternal(ASTNode ast) throws SemanticException {
...
    ctx.setExplain(true);
    ctx.setExplainLogical(logical);     // Create a semantic analyzer for the query
    ASTNode input = (ASTNode) ast.getChild(0);
    BaseSemanticAnalyzer sem = SemanticAnalyzerFactory.get(queryState, input);
    sem.analyze(input, ctx);
    sem.validate();     ctx.setResFile(ctx.getLocalTmpPath());
    List<Task<? extends Serializable>> tasks = sem.getAllRootTasks();
    if (tasks == null) {
      tasks = Collections.emptyList();
    }     FetchTask fetchTask = sem.getFetchTask();
    if (fetchTask != null) {
      // Initialize fetch work such that operator tree will be constructed.
      fetchTask.getWork().initializeForFetch(ctx.getOpContext());
    }     ParseContext pCtx = null;
    if (sem instanceof SemanticAnalyzer) {
      pCtx = ((SemanticAnalyzer)sem).getParseContext();
    }     boolean userLevelExplain = !extended
        && !formatted
        && !dependency
        && !logical
        && !authorize
        && (HiveConf.getBoolVar(ctx.getConf(), HiveConf.ConfVars.HIVE_EXPLAIN_USER) && HiveConf
            .getVar(conf, HiveConf.ConfVars.HIVE_EXECUTION_ENGINE).equals("tez"));
    ExplainWork work = new ExplainWork(ctx.getResFile(),
        pCtx,
        tasks,
        fetchTask,
        sem,
        extended,
        formatted,
        dependency,
        logical,
        authorize,
        userLevelExplain,
        ctx.getCboInfo());     work.setAppendTaskType(
        HiveConf.getBoolVar(conf, HiveConf.ConfVars.HIVEEXPLAINDEPENDENCYAPPENDTASKTYPES));     ExplainTask explTask = (ExplainTask) TaskFactory.get(work, conf);     fieldList = explTask.getResultSchema();
    rootTasks.add(explTask);
  }
```