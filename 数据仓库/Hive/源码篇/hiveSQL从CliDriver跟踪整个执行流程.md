## 前言

对接和使用hive的上层组件也很多了，hiveserver2，beeline，metastore，hivehook等，但没好好调试过hive，计划进行一次比较狠的补课，让自己对hive内部的很多实现有一个基础的认识，同时自定义实现一些模块。

本次是hive源码层的调试与实验的开坑，初步想象的过程有：hive执行大致全流程调试，序列化反序列化，执行计划，metastore，hivehook，udf，hiveserver2等，会分好几篇来写，算是学习整合记录。

## 准备工作

在hadoop与hive的部署调试篇完成本机部署与远程debug操作。

本次hive调试版本都为1.2.1。

## SHOW CODE

先借鉴一份整体的流程

```
Hive SQL - (Parser) -> AST - (Semantic Analyze) -> QB -  
(Logical Plan) -> Operator Tree - (Physical Plan) -> 
Task Tree - (Physical Optim) -> Task Tree

主要有三大块，SQL解析，逻辑执行计划，物理执行计划
```

找到入口类org.apache.hadoop.hive.hive.cli.CliDriver
断点打在其中的main函数的

> int ret = new CliDriver().run(args);

启动debug。

追踪到run函数，run中上面就是一些初始化类的操作
追踪到最后的executeDriver。

executeDriver 如果是初始化hive客户端时，会直接返回状态0。

如果是有命令执行的时候
executeDriver中主要是读取每一行，到分号为止并执行

> ret = cli.processLine(line, true); //line为执行sql

进入processLine，上面绑定重写了退出信号interruptSignal的处理，
下面真正进入执行的为

> ret = processCmd(command);

processCmd判断了一下sql是不是quit，source，!的情况，这边还是走他默认的流程。

> ret = processLocalCmd(cmd, proc, ss);

找到processLocalCmd的

> ret = qp.run(cmd).getResponseCode();

继续进入 run 的 runInternal, 找到

> ret = compileInternal(command);

进入compileInternal 的 compile
compile这边比较重要，重点看一下。

```
...
ASTNode tree = pd.parse(command, ctx); //SQL解析出AST树，这边用的是antlr3
...
SessionState.get().initTxnMgr(conf); // 初始化一个事务管理，记录这次query的信息的
...
hook.preAnalyze(hookCtx, tree); // 然后在analyze前后有两个hive hook的执行，如果有的话
sem.analyze(tree, ctx); //创建逻辑和物理执行计划，并优化，这里面的执行逻辑很多，后续再详细看
hook.postAnalyze(hookCtx, sem.getRootTasks());
...
```

然后继续跳回到runInternal，执行至，进入

> ret = execute();

然后执行

```
PRE_EXEC_HOOK //执行hook
DriverContext driverCxt = new DriverContext(ctx); //初始化运行容器
driverCxt.prepare(plan);
// Add root Tasks to runnable
for (Task<? extends Serializable> tsk : plan.getRootTasks()) {
    driverCxt.addToRunnable(tsk); //添加running任务，任务会进入一个队列
}
...
TaskRunner tskRun = driverCxt.pollFinished(); // poll已经完成了的任务, 加到hookContext
hookContext.addCompleteTask(tskRun);
...
// 如果中间exit，会触发ON_FAILURE_HOOK
... 
// 遍历子任务加到running
if (tsk.getChildTasks() != null) {
    for (Task<? extends Serializable> child : tsk.getChildTasks()) {
        if (DriverContext.isLaunchable(child)) {
            driverCxt.addToRunnable(child);
        }
    }
}
// 调用POST_EXEC_HOOK
// 然后计算了一波cpu是使用情况
plan.setDone(); //完成
```



 