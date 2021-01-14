[TOC]

## 执行计划进阶

前面我们介绍了执行计划基础，包括怎么看执行计划，执行计划中的各个operator的含义，其实我们知道一个SQL 从解析到执行是有一些的流程的，主要就是 SQL解析，逻辑执行计划，物理执行计划

```
Hive SQL - (Parser) -> AST - (Semantic Analyze) -> QB -  (Logical Plan) -> Operator Tree - (Physical Plan) -> Task Tree - (Physical Optim) -> Task Tree
```

### 从Driver的compile开始

定位到下面这行，进入

> sem.analyze(tree, ctx);

```
public void analyze(ASTNode ast, Context ctx) throws SemanticException {
    ...
    analyzeInternal(ast); //进入
}
```

继续进入analyzeInternal方法，这时已经定位到SemanticAnalyzer类了

```
void analyzeInternal(ASTNode ast, PlannerContext plannerCtx) throws SemanticException {
    // 1. Generate Resolved Parse tree from syntax tree
    LOG.info("Starting Semantic Analysis");
    if (!genResolvedParseTree(ast, plannerCtx)) { // 这边将每个AST节点转换为query block，跟进去看看
      return;
    }
    // 2. Gen OP Tree from resolved Parse Tree
    Operator sinkOp = genOPTree(ast, plannerCtx); //生成OperatorTree
    ...
    // 7. Perform Logical optimization
    // 进行了查询优化
    Optimizer optm = new Optimizer();
    optm.setPctx(pCtx);
    optm.initialize(conf);
    pCtx = optm.optimize();
}
boolean genResolvedParseTree(ASTNode ast, PlannerContext plannerCtx) throws SemanticException {
    ...
    // 4. continue analyzing from the child ASTNode.
    Phase1Ctx ctx_1 = initPhase1Ctx();
    preProcessForInsert(child, qb);
    if (!doPhase1(child, qb, ctx_1, plannerCtx)) { //这里面基本就是对于每种的转换逻辑，将空的qb传进去进行填充
      // if phase1Result false return
      return false;
    }
    ...
    getMetaData(qb); //从元数据获取上面explain中如下信息
                  //table: input，output，serde等
}
public boolean doPhase1(ASTNode ast, QB qb, Phase1Ctx ctx_1, PlannerContext plannerCtx)
      throws SemanticException {
    ...
    switch (ast.getToken().getType()) { //判断AST节点类型
        case HiveParser.TOK_SELECTDI:
        ...
        case HiveParser.TOK_SELECT:
        ...
        case HiveParser.TOK_WHERE:
        ...
        case HiveParser.TOK_INSERT_INTO:
        ...
        ...
    }
}
```

继续跟上面的genOPTree，跳转到genPlan

genPlan是一次如下的深度优先遍历生成树

```
private Operator genPlan(QB parent, QBExpr qbexpr) throws SemanticException {
    if (qbexpr.getOpcode() == QBExpr.Opcode.NULLOP) {
      boolean skipAmbiguityCheck = viewSelect == null && parent.isTopLevelSelectStarQuery();
      return genPlan(qbexpr.getQB(), skipAmbiguityCheck);
    }
    if (qbexpr.getOpcode() == QBExpr.Opcode.UNION) {
      Operator qbexpr1Ops = genPlan(parent, qbexpr.getQBExpr1());
      Operator qbexpr2Ops = genPlan(parent, qbexpr.getQBExpr2());

      return genUnionPlan(qbexpr.getAlias(), qbexpr.getQBExpr1().getAlias(),
          qbexpr1Ops, qbexpr.getQBExpr2().getAlias(), qbexpr2Ops);
    }
}

public Operator genPlan(QB qb) throws SemanticException {
    return genPlan(qb, false);
}

public Operator genPlan(QB qb, boolean skipAmbiguityCheck)
      throws SemanticException {
    for (String alias : qb.getSubqAliases()) {
      QBExpr qbexpr = qb.getSubqForAlias(alias);
      aliasToOpInfo.put(alias, genPlan(qb, qbexpr));
    }
    ...
}
```

### 优化器

上面也已经涉及到优化器optimizer，

### optimizer的主要功能

> （1）将多 multiple join 合并为一个 multi-way join；
> （2）对join、group-by 和自定义的 map-reduce 操作重新进行划分；
> （3）消减不必要的列；
> （4）在表扫描操作中推行使用断言（predicate）；
> （5）对于已分区的表，消减不必要的分区；
> （6）在抽样（sampling）查询中，消减不必要的桶。此外，优化器还能增加局部聚合操作用于处理大分组聚合（grouped aggregations）和 增加再分区操作用于处理不对称（skew）的分组聚合。

### 追踪Optimizer.initialize

```
public void initialize(HiveConf hiveConf) {
    // 往这里添加优化器，默认加下面两个，其他的根据conf配置添加
    transformations = new ArrayList<Transform>();

    // Add the additional postprocessing transformations needed if
    // we are translating Calcite operators into Hive operators.
    transformations.add(new HiveOpConverterPostProc());

    // Add the transformation that computes the lineage information.
    transformations.add(new Generator());
    ...
}

public ParseContext optimize() throws SemanticException {
    // 执行每个优化器Transform的transform信息进行
    // ParseContext的优化
    for (Transform t : transformations) {
        pctx = t.transform(pctx);
    }
    return pctx;
}
```

