[TOC]

## 前言

对于一般的网页上sql查询工具都会有一个功能【格式化】，对于hql的格式化市面上没有很好的直接使用的工具。

这边基于antlr4对hql的解析手动实现一个格式化的功能。

## 设计思路

对于一般比较固定的语句如create，show，drop，alter等采用直接对拿到节点并带格式用字符串拼接。

对于最复杂的select，因为其中包含很多嵌套，因此这边基本的想法有两种：

1. 因为visit树节点用的是深度优先，所以只要实现实现覆盖全部子节点的节点格式化字符并拼接起来，对于中间的嵌套，采用标记记录当前select的层级。
   比如默认层级为0，进入深一层select层级+1，然后继续递归子节点，在退出select后层级-1。
2. 采用替换法，首先遍历第一层的select，找到其中全部的子select并用startIndex做唯一标记，存储标记和层级，然后在到子select时判断标记是否存在存在则用替换法替换掉全局select结果中的待标记的select，比如select * from ({select1});

## 实现思路

这边采用的还是antlr4的visit模式，并用第二种方法实现一遍select的主要核心部分。

### 主SELECT部分

```
@Override
public Object visitSelect_stmt(HplsqlParser.Select_stmtContext ctx) {
    // 用全局标记判断是否为第一次进入，第一次进入也在结果字符串加上{SELECT%s}
    // 处理with子句
    // 遍历子select添加位置标记{SELECT%s},保存层级
    // 格式化每个子select
    // 格式化方式采用Optional判断是否存在该节点，然后拼接到当前select
    // 用当前格式化后的select替换掉标记
}
```

### SELECT字段

```
private String getSelectItem(HplsqlParser.Select_list_itemContext selectItem, Integer kong) {
    // 处理case when的情况
    // 否则直接截取源sql中select的字段的字符串
}
```

### FROM来源

```
private String getFromTableClause(HplsqlParser.From_table_clauseContext from_clauseContext, Integer kong) {
    // 处理子select
    // 处理table来源的情况
}
private String getFromJoin(HplsqlParser.From_join_clauseContext ctx, int kong) {
    // 处理from join的情况
}
```

### WHERE条件

```
private void makeWhereExpr(HplsqlParser.Bool_exprContext bool_exprContext) {
    // 处理where的条件
    // 需要用到递归，树如下
                     bool_expr
                         |
        bool_expr      operator     bool_expr
            |                           |
bool_expr.operator,bool_expr bool_expr,operator,bool_expr
}
```

### 关键字大小写处理

对于关键字的处理，本次不做详述，主要放在外面。
从lexer获取到全部定义的关键字，然后将sql切分，如果遇到关键字就替换。

> HplsqlLexer lexer = new HplsqlLexer(input);
> lexer.getAllTokens();

### sql注释

对于sql中存在注释的，默认会不解析到，因此格式化后的结果暂时无法还原注释。

## 结果展示

```
select id, name from (select id,name from (select id, name from user_table ut left join test_table tt on ut.id = tt.id)) where id > 1 and id < 3
SELECT
    id,
    name
FROM
(
    SELECT
        id,
        name
    FROM
    (
        SELECT
            id,
            name
        FROM
            user_table ut
        LEFT JOIN
            test_table tt
        ON ut.id = tt.id
    )
)
WHERE
    id > 1 AND
    id < 3
```

##  完整代码



```java
public class HiveSQLFormatter extends HplsqlBaseVisitor {
    private String sourceSQL;

    private String formattedSQL = "";

    private boolean firstSelect = true;

    public HiveSQLFormatter(String sql) {
        this.sourceSQL = sql;
    }

    // 用于从源字符串中截取，主要为了不继续深入，比如select的每个字段，会有计算和各个函数包裹等，就采用直接截取源字符串
    private String subSourceSql(ParserRuleContext parserRuleContext) {
        return sourceSQL.substring(
                parserRuleContext.getStart().getStartIndex(),
                parserRuleContext.getStop().getStopIndex() + 1);
    }

    //添加空占位，主要为了缩进
    private String addKongFormat(String s, Integer n) {
        return String.format("%" + (s.length() + n * 4) + "s", s);
    }

    // 保存某个select的层级，key为startIndex
    private Map<Integer, Integer> ceng = new HashMap<>();

    private List<String> boolExpr = new ArrayList<>();
    private List<String> boolExprOperator = new ArrayList<>();

    // 由于where多重条件判断是树状 -> (前面条件) 操作 (最后一个条件)，扩展左子树
    private void makeWhereExpr(HplsqlParser.Bool_exprContext bool_exprContext) {
        if(bool_exprContext.children.size() == 3 && bool_exprContext.bool_expr().size() == 2) {
            makeWhereExpr(bool_exprContext.bool_expr(0));
            boolExpr.add(sourceSQL.substring(bool_exprContext.bool_expr(1).getStart().getStartIndex(),
                    bool_exprContext.bool_expr(1).getStop().getStopIndex() + 1));
            boolExprOperator.add(bool_exprContext.bool_expr_logical_operator().getText());
        }else{
            boolExpr.add(sourceSQL.substring(bool_exprContext.getStart().getStartIndex(),
                    bool_exprContext.getStop().getStopIndex()+1));
        }
    }
    
    // select from 相关
    private String getFromTableClause(HplsqlParser.From_table_clauseContext from_clauseContext, Integer kong) {
        StringBuilder tmpSelect = new StringBuilder();
        Optional.of(from_clauseContext)
                .map(HplsqlParser.From_table_clauseContext::from_table_name_clause)
                .map(RuleContext -> addKongFormat(subSourceSql(RuleContext), kong + 1))
                .ifPresent(tmpSelect::append);
        // from 子select(
        Optional.of(from_clauseContext)
                .map(HplsqlParser.From_table_clauseContext::from_subselect_clause)
                .map(HplsqlParser.From_subselect_clauseContext::T_OPEN_P)
                .map(ParseTree -> addKongFormat(ParseTree.getText(), kong))
                .ifPresent(tmpSelect::append);
        // 添加子select标记
        Integer fromSubIndex = Optional.of(from_clauseContext)
                .map(HplsqlParser.From_table_clauseContext::from_subselect_clause)
                .map(HplsqlParser.From_subselect_clauseContext::select_stmt)
                .map(ParserRuleContext -> ParserRuleContext.getStart().getStartIndex()).orElse(null);
        if(fromSubIndex != null) {
            tmpSelect.append("\n{SELECT").append(fromSubIndex).append("}");
            ceng.put(fromSubIndex, kong + 1);
        }
        // )
        Optional.of(from_clauseContext)
                .map(HplsqlParser.From_table_clauseContext::from_subselect_clause)
                .map(HplsqlParser.From_subselect_clauseContext::T_CLOSE_P)
                .map(ParseTree -> "\n" + addKongFormat(ParseTree.getText(), kong))
                .ifPresent(tmpSelect::append);
        // from别名
        Optional.of(from_clauseContext)
                .map(HplsqlParser.From_table_clauseContext::from_subselect_clause)
                .map(HplsqlParser.From_subselect_clauseContext::from_alias_clause)
                .map(ParserRuleContext -> " " + subSourceSql(ParserRuleContext)).ifPresent(tmpSelect::append);
        return tmpSelect.toString();
    }

    //  select 字段相关
    private String getSelectItem(HplsqlParser.Select_list_itemContext selectItem, Integer kong) {
        AtomicReference<String> itemRes = new AtomicReference<>("");
        boolean isCase = Optional.of(selectItem)
                .map(HplsqlParser.Select_list_itemContext::expr)
                .map(HplsqlParser.ExprContext::expr_case)
                .map(HplsqlParser.Expr_caseContext::expr_case_searched)
                .map(expr_case_searchedContext -> {
                    StringBuilder tmpbuilder = new StringBuilder();
                    List<String> tmps = new ArrayList<>();
                    Optional.of(expr_case_searchedContext)
                            .map(HplsqlParser.Expr_case_searchedContext::T_CASE)
                            .map(ParseTree -> addKongFormat(ParseTree.getText() + "\n", kong + 1))
                            .map(tmpbuilder::append);
                    Optional.of(expr_case_searchedContext)
                            .map(HplsqlParser.Expr_case_searchedContext::T_WHEN)
                            .ifPresent(whenNodes -> {
                                for(int i = 0; i < whenNodes.size(); i++){
                                    tmps.add(addKongFormat(expr_case_searchedContext.T_WHEN(i).getText() + " " +
                                            subSourceSql(expr_case_searchedContext.bool_expr(i)) + " " +
                                            expr_case_searchedContext.T_THEN(i).getText() + " " +
                                            subSourceSql(expr_case_searchedContext.expr(i)), kong + 2));
                                }
                                tmpbuilder.append(StringUtils.join(tmps.toArray(), "\n"));
                            });
                    Optional.of(expr_case_searchedContext).map(HplsqlParser.Expr_case_searchedContext::T_ELSE)
                            .map(ParseTree -> "\n" + addKongFormat(ParseTree.getText() +
                                    " " + expr_case_searchedContext.expr(expr_case_searchedContext.expr().size()-1).getText(), kong + 2))
                            .map(tmpbuilder::append);
                    Optional.of(expr_case_searchedContext)
                            .map(HplsqlParser.Expr_case_searchedContext::T_END)
                            .map(ParseTree -> "\n" + addKongFormat(ParseTree.getText(), kong + 1))
                            .map(tmpbuilder::append);
                    Optional.of(selectItem)
                            .map(HplsqlParser.Select_list_itemContext::select_list_alias)
                            .map(select_list_aliasContext -> "\n" + addKongFormat(subSourceSql(select_list_aliasContext), kong + 1))
                            .map(tmpbuilder::append);
                    itemRes.set(tmpbuilder.toString());
                    return true;
                }).orElse(false);
        if (!isCase) {
            Optional.of(selectItem)
                    .ifPresent(select_list_itemContext -> itemRes.set(addKongFormat(subSourceSql(selectItem), kong + 1)));
        }
        return itemRes.get();
    }

    private String getLateralView(HplsqlParser.Lateral_clause_itemContext ctx, int kong) {
        StringBuilder lateralView = new StringBuilder();
        Optional.of(ctx)
                .map(HplsqlParser.Lateral_clause_itemContext::T_LATERAL)
                .map(lateralView::append);
        lateralView.append(" ");
        Optional.of(ctx)
                .map(HplsqlParser.Lateral_clause_itemContext::T_VIEW)
                .map(lateralView::append);
        lateralView.append("\n");
        Optional.of(ctx)
                .map(HplsqlParser.Lateral_clause_itemContext::expr)
                .map(ParserRuleContext -> addKongFormat(subSourceSql(ParserRuleContext), kong+1))
                .map(lateralView::append);
        lateralView.append(" ");
        Optional.of(ctx)
                .map(HplsqlParser.Lateral_clause_itemContext::ident)
                .map(RuleContext::getText)
                .map(lateralView::append);
        Optional.of(ctx)
                .map(HplsqlParser.Lateral_clause_itemContext::T_AS)
                .map(ParserRuleContext -> "\n" + ParserRuleContext.getText())
                .map(lateralView::append);
        Optional.of(ctx)
                .map(HplsqlParser.Lateral_clause_itemContext::lateral_clause_alias)
                .map(ParserRuleContext -> " " + subSourceSql(ParserRuleContext))
                .map(lateralView::append);
        return lateralView.toString();
    }

    private String getFromJoin(HplsqlParser.From_join_clauseContext ctx, int kong) {
        StringBuilder fromjoinSQL = new StringBuilder();
        Optional.of(ctx)
                .map(HplsqlParser.From_join_clauseContext::from_join_type_clause)
                .map(from_join_type_clauseContext -> "\n" + addKongFormat(subSourceSql(ctx.from_join_type_clause()), kong) + "\n"
                        + getFromTableClause(ctx.from_table_clause(), kong) + "\n"
                        + addKongFormat(ctx.T_ON().getText(), kong) + " "
                        + subSourceSql(ctx.bool_expr()))
                .map(fromjoinSQL::append);
        Optional.of(ctx)
                .map(HplsqlParser.From_join_clauseContext::T_COMMA)
                .map(ParserRuleContext -> ParserRuleContext.getText() + "\n"
                        + getFromTableClause(ctx.from_table_clause(), kong)
                ).map(fromjoinSQL::append);
        return fromjoinSQL.toString();
    }
    
    // 使用替换模式替换{SELECTN}
    @Override
    public Object visitSelect_stmt(HplsqlParser.Select_stmtContext ctx) {
        int selectSize = ctx.fullselect_stmt().fullselect_stmt_item().size();
        // 第一次进入添加位置标记(节点在原字符串startIndex作为唯一标志)
        int gstartIndex = ctx.getStart().getStartIndex();
        String thisSelect = String.format("{SELECT%s}", gstartIndex);
        if(!formattedSQL.contains(thisSelect) && firstSelect){
            formattedSQL += thisSelect;
            firstSelect = false;
        }
        StringBuilder fullSelect = new StringBuilder();
        // with语句
        Optional.of(ctx)
                .map(HplsqlParser.Select_stmtContext::cte_select_stmt)
                .map(HplsqlParser.Cte_select_stmtContext::T_WITH)
                .map(ParserRuleContext -> ParserRuleContext.getText() + " ")
                .map(fullSelect::append);
        Optional.of(ctx)
                .map(HplsqlParser.Select_stmtContext::cte_select_stmt)
                .map(HplsqlParser.Cte_select_stmtContext::cte_select_stmt_item)
                .map(cte_select_stmt_itemContexts -> StringUtils.join(cte_select_stmt_itemContexts.stream().map(
                        item -> item.ident().getText() + " " +
                                item.T_AS().getText() + "\n" +
                                item.T_OPEN_P().getText() + "" +
                                String.format("\n{SELECT%s}", item.select_stmt().getStart().getStartIndex()) + "\n" +
                                item.T_CLOSE_P().getText()
                ).toArray(), ",\n")).map(fullSelect::append);
        // 遍历子select添加位置标记
        Integer gkong = ceng.get(gstartIndex) == null ? 0: ceng.get(gstartIndex);
        for(int i = 0; i < selectSize; i++) {
            Integer startIndex = ctx.fullselect_stmt().fullselect_stmt_item(i).subselect_stmt().getStart().getStartIndex();
            fullSelect.append(String.format("{SELECT%s}", startIndex));
            ceng.put(startIndex, gkong);
            if(i < selectSize - 1) {
                HplsqlParser.Fullselect_set_clauseContext clauseContext = ctx.fullselect_stmt().fullselect_set_clause(i);
                fullSelect.append("\n").append(addKongFormat(subSourceSql(clauseContext), gkong)).append("\n");
            }
        }
        // 格式化每个子select并替换标记
        formattedSQL = formattedSQL.replace(String.format("{SELECT%s}", ctx.getStart().getStartIndex()), fullSelect.toString());
        for(int i = 0; i < selectSize; i++) {
            HplsqlParser.Subselect_stmtContext subSelect = ctx.fullselect_stmt().fullselect_stmt_item(i).subselect_stmt();
            Integer thisIndex = subSelect.getStart().getStartIndex();
            Integer kong = ceng.get(thisIndex) == null ? 0: ceng.get(thisIndex);
            StringBuilder tmpSelect = new StringBuilder();
            // select
            tmpSelect.append(addKongFormat(
                    subSelect.T_SELECT().getText(),
                    kong)).append("\n");
            Optional.of(subSelect).map(HplsqlParser.Subselect_stmtContext::select_list)
                    .map(HplsqlParser.Select_listContext::select_list_set)
                    .map(select_list_setContext -> addKongFormat(subSourceSql(select_list_setContext)+"\n", kong+1))
                    .map(tmpSelect::append);
            Optional.of(subSelect).map(HplsqlParser.Subselect_stmtContext::select_list)
                    .map(HplsqlParser.Select_listContext::select_list_item)
                    .map(select_list_itemContexts ->
                            StringUtils.join(select_list_itemContexts.stream().map(
                                    item -> getSelectItem(item, kong)).toArray(), ",\n") + "\n").map(tmpSelect::append);
            // from
            Optional.of(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::from_clause)
                    .map(HplsqlParser.From_clauseContext::T_FROM)
                    .map(ParserRuleContext -> addKongFormat(ParserRuleContext.getText() + "\n", kong))
                    .map(tmpSelect::append);
            // from 表
            Optional.of(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::from_clause)
                    .map(HplsqlParser.From_clauseContext::from_table_clause)
                    .map(from_table_clauseContext -> getFromTableClause(from_table_clauseContext, kong))
                    .map(tmpSelect::append);
            // join语句
            Optional.of(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::from_clause)
                    .map(HplsqlParser.From_clauseContext::from_join_clause)
                    .map(from_join_clauseContexts -> from_join_clauseContexts.size() > 0 ?
                            StringUtils.join(from_join_clauseContexts.stream().map(
                                    item -> getFromJoin(item, kong)).toArray(), ""): "")
                    .ifPresent(tmpSelect::append);
            Optional.of(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::lateral_clause)
                    .map(HplsqlParser.Lateral_clauseContext::lateral_clause_item)
                    .map(lateral_clause_itemContexts -> "\n" + StringUtils.join(lateral_clause_itemContexts.stream().map(
                            item -> getLateralView(item, kong)).toArray(), "\n"))
                    .map(tmpSelect::append);
            // where
            Optional.of(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::where_clause)
                    .map(HplsqlParser.Where_clauseContext::T_WHERE)
                    .map(ParseTree -> "\n" + addKongFormat(ParseTree.getText() + "\n", kong))
                    .ifPresent(tmpSelect::append);
            // where条件
            Optional.of(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::where_clause)
                    .ifPresent(ParserRuleContext -> {
                        makeWhereExpr(ParserRuleContext.bool_expr());
                        List<String> result = new ArrayList<>();
                        for(int t = 0; t < boolExpr.size(); t++) {
                            if(t == boolExpr.size() -1){
                                result.add(boolExpr.get(t));
                            }else{
                                result.add(boolExpr.get(t) + " " + boolExprOperator.get(t));
                            }
                        }
                        boolExpr = new ArrayList<>();
                        boolExprOperator = new ArrayList<>();
                        tmpSelect.append(StringUtils.join(result.stream().map(item -> addKongFormat(item, kong + 1)).toArray(), "\n"));
                    });
            // group by
            Optional.of(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::group_by_clause)
                    .map(ParserRuleContext -> "\n" + addKongFormat(ParserRuleContext.T_GROUP().getText() + " " + ParserRuleContext.T_BY().getText(), kong))
                    .ifPresent(tmpSelect::append);
            // group by 字段
            Optional.of(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::group_by_clause)
                    .map(HplsqlParser.Group_by_clauseContext::expr)
                    .map(exprContexts -> "\n" + addKongFormat(StringUtils.join(exprContexts.stream().map(this::subSourceSql).toArray(), ", "), kong + 1))
                    .ifPresent(tmpSelect::append);
            // having 语句
            Optional.of(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::having_clause)
                    .map(having_clauseContext -> "\n" + addKongFormat(subSourceSql(having_clauseContext), kong))
                    .ifPresent(tmpSelect::append);
            // order by
            Optional.of(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::order_by_clause)
                    .map(ParserRuleContext -> "\n" + addKongFormat(ParserRuleContext.T_ORDER() + " " + ParserRuleContext.T_BY(), kong))
                    .ifPresent(tmpSelect::append);
            // order by 字段
            Optional.of(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::order_by_clause)
                    .map(ParserRuleContext -> "\n" + addKongFormat(StringUtils.join(ParserRuleContext.expr().stream().map(
                            RuleContext::getText).toArray(), ", "), kong + 1))
                    .ifPresent((tmpSelect::append));
            // order by 参数
            Optional.of(subSelect).map(HplsqlParser.Subselect_stmtContext::order_by_clause)
                    .map(HplsqlParser.Order_by_clauseContext::T_ASC)
                    .map(ParserRuleContext -> ParserRuleContext.size() > 0 ? "\n" + ParserRuleContext.get(0).getText() : "").ifPresent(tmpSelect::append);
            Optional.of(subSelect).map(HplsqlParser.Subselect_stmtContext::order_by_clause)
                    .map(HplsqlParser.Order_by_clauseContext::T_DESC)
                    .map(ParserRuleContext -> ParserRuleContext.size() > 0 ? "\n" + ParserRuleContext.get(0).getText() : "").ifPresent(tmpSelect::append);
            // select 参数
            Optional.of(subSelect).map(HplsqlParser.Subselect_stmtContext::select_options)
                    .map(ParserRuleContext -> "\n" + addKongFormat(subSourceSql(ParserRuleContext), kong))
                    .ifPresent(tmpSelect::append);
            formattedSQL = formattedSQL.replace(String.format("{SELECT%s}", thisIndex), tmpSelect.toString());
        }
        return super.visitSelect_stmt(ctx);
    }
}
```