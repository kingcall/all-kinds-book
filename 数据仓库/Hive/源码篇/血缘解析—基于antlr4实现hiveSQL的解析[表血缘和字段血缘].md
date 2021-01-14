## 前言



关于 HiveSQL 血缘，一般表示的就是 hive 数据仓库中所有表和字段的来源流向关系。它的解析是十分必要的，一方面数仓建表的时候有时只会定义 SQL 任务不会特别关注到任务之间的关系，对于查看的数据也不容易追溯两层以上或以下的数据来源和去向。

有了血缘就可以对离线任务执行的先后关系作出一定规范，可以做数据来源链路的分析，数据的上卷下钻，数仓直接的数据建模等。

## 实现思路

一般来说比较直接的实现方式是 hivehook 的 LineageLogger，但直接用也有比较明显麻烦的地方，一个是借用了 hive 自带的 antlr3 的 hql 解析，如果有部分语法不满足，去修改解析文件会造成不可控影响；另一个用 hivehook 实现后的迭代测试发布等都是一个比较麻烦的过程，出错了也很难定位问题所在。

这边就考虑用 antlr4 配合 hive 内部的 Hplsql.g4 直接实现一个血缘的解析。实现方式还是 visit 模式。

### 表血缘

首先表血缘是比较之间简单的，比如对于一个 insert 来说，目标表永远只有一个，来源表是 select 中所有 from 的真实表。

### 字段血缘

对于字段血缘实现会麻烦一点，因为要将每个结果字段的层层关系找到并最后对应上真实表的字段，可能中间还会有多个字段计算为一个字段，一个字段于下层多个字段有血缘，还会有表别名，字段别名的干扰。

这边最后的考虑是将每个 select 剥离出来存成一个 object，其中包括来源表(来源子 select 则为 null)，select 字段，父 select 的 Index(第一层则为 null)。在解析完成后所有 select 的 object 存为一个数组，然后逐个对最外层的字段进行溯源找到真实的来源表。

## SHOW CODE

### 表血缘

首先定义好结构

```
// 表名的结构
public class TableNameModel {
    private String dbName;
    private String tableName;

    public static String dealNameMark(String name) {
        if(name.startsWith("`") && name.endsWith("`")) {
            return name.substring(1, name.length()-1);
        }else{
            return name;
        }
    }

    public static TableNameModel parseTableName(String tableName) {
        TableNameModel tableNameModel = new TableNameModel();
        String[] splitTable = tableName.split("\\.");
        if(splitTable.length == 2) {
            tableNameModel.setDbName(splitTable[0]);
            tableNameModel.setTableName(splitTable[1]);
        }else if(splitTable.length == 1) {
            tableNameModel.setTableName(splitTable[0]);
        }
        return tableNameModel;
    }
}

// 表血缘结构，对单个sql
public class HiveTableLineageModel {
    // 输出的表名
    private TableNameModel outputTable;
    // 输入的表名列表
    private HashSet<TableNameModel> inputTables;
}
```

表血缘主要过程，监听 insert 语句

```
public class HiveSQLTableLineage extends HplsqlBaseVisitor {

    private TableNameModel outputTable;
    private HashSet<TableNameModel> inputTables = new HashSet<>();

    // visitInsert获取insert的table_name节点，作为目标输出表
    @Override
    public Object visitInsert_stmt(HplsqlParser.Insert_stmtContext ctx) {
         outputTable = Optional.ofNullable(ctx)
                .map(HplsqlParser.Insert_stmtContext::table_name)
                .map(RuleContext::getText)
                .map(TableNameModel::parseTableName)
                .orElse(null);
        return super.visitInsert_stmt(ctx);
    }

    // 获取from真实表，加到来源表的Set里
    @Override
    public Object visitFrom_table_clause(HplsqlParser.From_table_clauseContext ctx) {
        Optional.ofNullable(ctx)
                .map(HplsqlParser.From_table_clauseContext::from_table_name_clause)
                .map(RuleContext::getText)
                .map(TableNameModel::parseTableName)
                .map(inputTables::add);
        return super.visitFrom_table_clause(ctx);
    }

    public HiveTableLineageModel getTableLineage() {
        HiveTableLineageModel hiveTableLineageModel = new HiveTableLineageModel();
        hiveTableLineageModel.setOutputTable(outputTable);
        hiveTableLineageModel.setInputTables(inputTables);
        return hiveTableLineageModel;
    }

}
```

### 字段血缘

定义结构

```
// 字段名
public class FieldNameModel {
    private String dbName;
    private String tableName;
    private String fieldName;
}

// 带过程的表字段
public class FieldNameWithProcessModel {
    private String dbName;
    private String tableName;
    private String fieldName;
    private String process;
}

// 解析单个select中存储字段和别名
// 如：select a+b as c from table;
// 存储数据为 fieldNames:[a,b] alias:c process:a+b
public class HiveFieldLineageSelectItemModel {
    private Set<String> fieldNames;
    private String alias;
    private String process;
}

// 解析单个select后的结果
public class HiveFieldLineageSelectModel {
    Integer id; // index
    Integer parentId; // 父id，第一层select为null
    TableNameModel fromTable; // 来源表，来源子select则为null
    String tableAlias; // 表别名
    List<HiveFieldLineageSelectItemModel> selectItems; // select字段
}

// 血缘结果结构
public class HiveFieldLineageModel {
    private FieldNameModel targetField; // 目标字段
    private HashSet<FieldNameModel> sourceFields; // 来源字段列表
}
```

字段血缘主要过程,主要针对的是 insert 语句，
但一般的 select 也是可以用的，因为是把最外层 select 的字段作为结果字段,
有一个限制是中间不能有 select * 这种操作，因为目前不连接元数据库，就无法获得*对应的字段。
中间也记录了字段流转的计算过程，理应是一个数组，取了最长一个，这边比较不稳定。

```
public class HiveSQLFieldLineage extends HplsqlBaseVisitor {

    private TableNameModel outputTable;

    private HashMap<String, HiveFieldLineageSelectModel> hiveFieldSelects = new LinkedHashMap<>();

    private Map<Integer, String> selectParentKeyMap = new HashMap<>();

    private String thisSelectId;

    private String sourceSQL;

    // for select Item
    private HiveFieldLineageSelectItemModel selectItemModel;
    private List<HiveFieldLineageSelectItemModel> selectFields = new ArrayList<>();
    private Boolean startSelectItem = false;

    public HiveSQLFieldLineage(String sql) {
        this.sourceSQL = sql;
    }

    private String subSourceSql(ParserRuleContext parserRuleContext) {
        return sourceSQL.substring(
                parserRuleContext.getStart().getStartIndex(),
                parserRuleContext.getStop().getStopIndex() + 1);
    }

    /**
     * insert解析结果表
     */
    @Override
    public Object visitInsert_stmt(HplsqlParser.Insert_stmtContext ctx) {
        outputTable = Optional.ofNullable(ctx)
                .map(HplsqlParser.Insert_stmtContext::table_name)
                .map(RuleContext::getText)
                .map(TableNameModel::parseTableName)
                .orElse(null);
        return super.visitInsert_stmt(ctx);
    }

    /**
     * 解析select每个selectItem里用到字段
     */
    @Override
    public Object visitExpr(HplsqlParser.ExprContext ctx) {
        if(startSelectItem) {
            Optional.ofNullable(ctx)
                    .map(HplsqlParser.ExprContext::expr_atom)
                    .map(HplsqlParser.Expr_atomContext::ident)
                    .map(ParseTree::getText)
                    .ifPresent(s -> {
                        if(!StringUtils.isNumeric(s)) {
                            selectItemModel.getFieldNames().add(TableNameModel.dealNameMark(s));
                        }
                    });
        }
        return super.visitExpr(ctx);
    }

    /**
     * selectItem 获取别名名，初始化selectItem存相关字段的fieldNames
     */
    @Override
    public Object visitSelect_list_item(HplsqlParser.Select_list_itemContext ctx) {
        startSelectItem = true;
        selectItemModel = new HiveFieldLineageSelectItemModel();
        selectItemModel.setFieldNames(new HashSet<>());
        Optional.ofNullable(ctx)
                .map(HplsqlParser.Select_list_itemContext::expr)
                .map(this::subSourceSql)
                .ifPresent(selectItemModel::setProcess);
        Optional.ofNullable(ctx)
                .map(HplsqlParser.Select_list_itemContext::select_list_alias)
                .map(HplsqlParser.Select_list_aliasContext::ident)
                .map(RuleContext::getText)
                .ifPresent(selectItemModel::setAlias);
        Object visit = super.visitSelect_list_item(ctx);
        selectFields.add(selectItemModel);
        return visit;
    }

    /**
     * from语句，处理于所有selectItem结束
     * 对上面解析出的字段名中的表别名进行处理 如t0.field
     */
    @Override
    public Object visitFrom_clause(HplsqlParser.From_clauseContext ctx) {
        startSelectItem = false;
        HashMap<String, List<HiveFieldLineageSelectItemModel>> fieldItems = new HashMap<>();
        for(HiveFieldLineageSelectItemModel item: selectFields) {
            HashMap<String, HashSet<String>> aliasSet = new HashMap<>();
            for(String field: item.getFieldNames()) {
                String[] sp = field.split("\\.");
                if(sp.length == 2){
                    String key = thisSelectId + "_" + sp[0];
                    aliasSet.computeIfAbsent(key, t -> new HashSet<>());
                    aliasSet.get(key).add(sp[1]);
                }else if(sp.length == 1){
                    boolean flat = true;
                    for(String k: selectParentKeyMap.values()) {
                        if(k.startsWith(thisSelectId + "_")) {
                            aliasSet.computeIfAbsent(k, t -> new HashSet<>());
                            aliasSet.get(k).add(sp[0]);
                            flat=false;
                        }
                    }
                    if(flat) {
                        String key = thisSelectId + "_";
                        aliasSet.computeIfAbsent(key, t -> new HashSet<>());
                        aliasSet.get(key).add(sp[0]);
                    }
                }
            }
            for(String key: aliasSet.keySet()) {
                fieldItems.computeIfAbsent(key, k -> new ArrayList<>());
                HiveFieldLineageSelectItemModel selectItemModel = new HiveFieldLineageSelectItemModel();
                selectItemModel.setFieldNames(aliasSet.get(key));
                selectItemModel.setAlias(item.getAlias());
                selectItemModel.setProcess(item.getProcess());
                if(selectItemModel.getFieldNames().size() == 1 && selectItemModel.getAlias() == null) {
                    selectItemModel.setAlias(selectItemModel.getFieldNames().iterator().next());
                }
                fieldItems.get(key).add(selectItemModel);
            }
        }
        for(String key: fieldItems.keySet()) {
            if(hiveFieldSelects.get(key) != null) {
                hiveFieldSelects.get(key).setSelectItems(fieldItems.get(key));
            }
        }
        return super.visitFrom_clause(ctx);
    }

    /**
     * 进入select前
     * 解析每个select存信息并另存父子关系
     * 父子来源于from subSelect, join subSelect
     */
    @Override
    public Object visitSelect_stmt(HplsqlParser.Select_stmtContext ctx) {
        List<HplsqlParser.Fullselect_stmt_itemContext> selectItems  = ctx.fullselect_stmt().fullselect_stmt_item();
        for(HplsqlParser.Fullselect_stmt_itemContext selectItem: selectItems) {
            HiveFieldLineageSelectModel hiveFieldLineageSelectModel = new HiveFieldLineageSelectModel();
            Integer thisId = selectItem.getStart().getStartIndex();
            HplsqlParser.Subselect_stmtContext subSelect = selectItem.subselect_stmt();
            HplsqlParser.From_table_name_clauseContext fromTableNameClause = Optional.ofNullable(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::from_clause)
                    .map(HplsqlParser.From_clauseContext::from_table_clause)
                    .map(HplsqlParser.From_table_clauseContext::from_table_name_clause)
                    .orElse(null);
            Optional.ofNullable(fromTableNameClause)
                    .map(HplsqlParser.From_table_name_clauseContext::table_name)
                    .map(RuleContext::getText)
                    .map(TableNameModel::parseTableName)
                    .ifPresent(hiveFieldLineageSelectModel::setFromTable);
            Optional.ofNullable(fromTableNameClause)
                    .map(HplsqlParser.From_table_name_clauseContext::from_alias_clause)
                    .map(HplsqlParser.From_alias_clauseContext::ident)
                    .map(RuleContext::getText)
                    .ifPresent(hiveFieldLineageSelectModel::setTableAlias);

            Optional.ofNullable(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::from_clause)
                    .map(HplsqlParser.From_clauseContext::from_table_clause)
                    .map(HplsqlParser.From_table_clauseContext::from_subselect_clause)
                    .map(HplsqlParser.From_subselect_clauseContext::from_alias_clause)
                    .map(RuleContext::getText)
                    .ifPresent(hiveFieldLineageSelectModel::setTableAlias);

            String alias = hiveFieldLineageSelectModel.getTableAlias();
            String thisKey = String.format("%s_%s", thisId, alias == null ? "": alias);
            hiveFieldLineageSelectModel.setId(thisKey + "");
            hiveFieldLineageSelectModel.setParentId(selectParentKeyMap.get(thisId));
            hiveFieldLineageSelectModel.setSelectItems(new ArrayList<>());
            hiveFieldSelects.put(thisKey, hiveFieldLineageSelectModel);

            Optional.ofNullable(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::from_clause)
                    .map(HplsqlParser.From_clauseContext::from_table_clause)
                    .map(HplsqlParser.From_table_clauseContext::from_subselect_clause)
                    .map(HplsqlParser.From_subselect_clauseContext::select_stmt)
                    .map(HplsqlParser.Select_stmtContext::fullselect_stmt)
                    .map(HplsqlParser.Fullselect_stmtContext::fullselect_stmt_item)
                    .ifPresent(subSelects ->
                            subSelects.forEach(item ->
                                    selectParentKeyMap.put(item.getStart().getStartIndex(), thisKey)));

            List<HplsqlParser.From_join_clauseContext> fromJoinClauses = Optional.ofNullable(subSelect)
                    .map(HplsqlParser.Subselect_stmtContext::from_clause)
                    .map(HplsqlParser.From_clauseContext::from_join_clause)
                    .orElse(new ArrayList<>());
            for(HplsqlParser.From_join_clauseContext fromJoinClauseContext: fromJoinClauses) {
                HiveFieldLineageSelectModel joinSelect = new HiveFieldLineageSelectModel();
                Optional.ofNullable(fromJoinClauseContext)
                        .map(HplsqlParser.From_join_clauseContext::from_table_clause)
                        .map(HplsqlParser.From_table_clauseContext::from_table_name_clause)
                        .map(HplsqlParser.From_table_name_clauseContext::table_name)
                        .map(RuleContext::getText)
                        .map(TableNameModel::parseTableName)
                        .ifPresent(joinSelect::setFromTable);
                Optional.ofNullable(fromJoinClauseContext)
                        .map(HplsqlParser.From_join_clauseContext::from_table_clause)
                        .map(HplsqlParser.From_table_clauseContext::from_table_name_clause)
                        .map(HplsqlParser.From_table_name_clauseContext::from_alias_clause)
                        .map(HplsqlParser.From_alias_clauseContext::ident)
                        .map(RuleContext::getText)
                        .ifPresent(joinSelect::setTableAlias);

                Optional.ofNullable(fromJoinClauseContext)
                        .map(HplsqlParser.From_join_clauseContext::from_table_clause)
                        .map(HplsqlParser.From_table_clauseContext::from_subselect_clause)
                        .map(HplsqlParser.From_subselect_clauseContext::from_alias_clause)
                        .map(RuleContext::getText)
                        .ifPresent(joinSelect::setTableAlias);

                String jalias = joinSelect.getTableAlias();
                String jkey = String.format("%s_%s", thisId, jalias == null ? "": jalias);
                joinSelect.setId(jkey);
                joinSelect.setParentId(selectParentKeyMap.get(thisId));
                joinSelect.setSelectItems(new ArrayList<>());
                hiveFieldSelects.put(jkey, joinSelect);

                Optional.ofNullable(fromJoinClauseContext)
                        .map(HplsqlParser.From_join_clauseContext::from_table_clause)
                        .map(HplsqlParser.From_table_clauseContext::from_subselect_clause)
                        .map(HplsqlParser.From_subselect_clauseContext::select_stmt)
                        .map(HplsqlParser.Select_stmtContext::fullselect_stmt)
                        .map(HplsqlParser.Fullselect_stmtContext::fullselect_stmt_item)
                        .ifPresent(subSelects ->
                                subSelects.forEach(item ->
                                        selectParentKeyMap.put(item.getStart().getStartIndex(), jkey)));
            }
        }
        return super.visitSelect_stmt(ctx);
    }

    /**
     * 处理每个子select进入前，
     * 初始化selectItem相关的变量
     */
    @Override
    public Object visitSubselect_stmt(HplsqlParser.Subselect_stmtContext ctx) {
        thisSelectId = ctx.getStart().getStartIndex() + "";
        selectFields = new ArrayList<>();
        return super.visitSubselect_stmt(ctx);
    }

    private List<HiveFieldLineageSelectModel> hiveFieldSelectList = new ArrayList<>();

    /**
     * 转换HashMap存储为List
     */
    private void transSelectToList() {
        for(String key: hiveFieldSelects.keySet()) {
            hiveFieldSelectList.add(hiveFieldSelects.get(key));
        }
    }

    /**
     * 获取目标字段
     * 也就是parentId为null的最外层select的字段别名
     */
    private List<FieldNameModel> getTargetFields() {
        List<List<String>> items = hiveFieldSelectList.stream()
                .filter(item -> item.getParentId() == null)
                .map(HiveFieldLineageSelectModel::getSelectItems)
                .map(fields -> fields.stream()
                        .map(HiveFieldLineageSelectItemModel::getAlias)
                        .collect(Collectors.toList()))
                .collect(Collectors.toList());
        List<String> res = new ArrayList<>();
        for(List<String> item: items) {
            res.addAll(item);
        }
        res = res.stream().distinct().collect(Collectors.toList());
        List<FieldNameModel> fieldNameModels = new ArrayList<>();
        for(String i: res) {
            FieldNameModel fieldNameModel = new FieldNameModel();
            if(outputTable != null) {
                fieldNameModel.setDbName(outputTable.getDbName());
                fieldNameModel.setTableName(outputTable.getTableName());
            }
            fieldNameModel.setFieldName(i);
            fieldNameModels.add(fieldNameModel);
        }
        return fieldNameModels;
    }

    private HashSet<FieldNameWithProcessModel> sourceFields;
    private String fieldProcess = "";

    /**
     * 递归按每个字段从外到内寻找每个字段的来源
     * 逻辑为最外的字段别名，父id -> 匹配子id别名 ->
     * -> 如果是来源是表，存储，如果来源是子select，继续递归
     */
    private void findFieldSource(String targetField, String parentId) {
        hiveFieldSelectList.forEach(select -> {
            if((parentId == null && select.getParentId() == null) ||
                    (select.getParentId() != null && select.getParentId().equals(parentId))) {
                if(select.getSelectItems() != null) {
                    if(select.getFromTable() == null) {
                        select.getSelectItems().forEach(selectItem -> {
                            if(selectItem.getAlias().equals(targetField)) {
                                if(selectItem.getProcess().length() > fieldProcess.length()) {
                                    fieldProcess = selectItem.getProcess();
                                }
                                for(String field: selectItem.getFieldNames()){
                                    findFieldSource(field, select.getId());
                                }
                            }
                        });
                    }else{
                        select.getSelectItems().forEach(selectItem -> {
                            if(selectItem.getAlias().equals(targetField)) {
                                if(selectItem.getProcess().length() > fieldProcess.length()) {
                                    fieldProcess = selectItem.getProcess();
                                }
                                for(String field: selectItem.getFieldNames()){
                                    FieldNameWithProcessModel fieldNameWithProcessModel = new FieldNameWithProcessModel();
                                    fieldNameWithProcessModel.setDbName(select.getFromTable().getDbName());
                                    fieldNameWithProcessModel.setTableName(select.getFromTable().getTableName());
                                    fieldNameWithProcessModel.setFieldName(field);
                                    fieldNameWithProcessModel.setProcess(fieldProcess);
                                    sourceFields.add(fieldNameWithProcessModel);
                                }
                            }
                        });
                    }
                }
            }
        });
    }

    /**
     * 获取字段血缘列表
     */
    public List<HiveFieldLineageModel> getHiveFieldLineage() {
        transSelectToList();
        List<FieldNameModel> targetFields = getTargetFields();
        List<HiveFieldLineageModel> hiveFieldLineageModelList = new ArrayList<>();
        for(FieldNameModel targetField: targetFields) {
            HiveFieldLineageModel hiveFieldLineageModel = new HiveFieldLineageModel();
            hiveFieldLineageModel.setTargetField(targetField);
            sourceFields = new HashSet<>();
            fieldProcess = "";
            findFieldSource(targetField.getFieldName(), null);
            hiveFieldLineageModel.setSourceFields(sourceFields);
            hiveFieldLineageModelList.add(hiveFieldLineageModel);
        }
        return hiveFieldLineageModelList;
    }

    /**
     * 获取sql解析处理后的结果
     */
    public HashMap<String, HiveFieldLineageSelectModel> getHiveFieldSelects() {
        return hiveFieldSelects;
    }
}
```

## 效果展示

举一个简单的 sql

```
INSERT INTO TABLE db_test.table_result
SELECT
    t1.id,
    t2.name
FROM
(
    SELECT
        id1 + id2 AS id
    FROM
        db_test.table1
) t1
LEFT JOIN
(
    SELECT
        id,
        name
    FROM
    (
        SELECT
            id,
            sourcename AS name
        FROM
            db_test.table2
    )
) t2
ON t1.id=t2.id
```

解析后的表血缘

```
{
    "inputTables": [
        {
            "dbName": "db_test",
            "tableName": "table2"
        },
        {
            "dbName": "db_test",
            "tableName": "table1"
        }
    ],
    "outputTable": {
        "dbName": "db_test",
        "tableName": "table_result"
    }
}
```

解析后的字段血缘

```
[
    {
        "sourceFields": [
            {
                "dbName": "db_test",
                "fieldName": "id1",
                "process": "id1 + id2",
                "tableName": "table1"
            },
            {
                "dbName": "db_test",
                "fieldName": "id2",
                "process": "id1 + id2",
                "tableName": "table1"
            }
        ],
        "targetField": {
            "dbName": "db_test",
            "fieldName": "id",
            "tableName": "table_result"
        }
    },
    {
        "sourceFields": [
            {
                "dbName": "db_test",
                "fieldName": "sourcename",
                "process": "sourcename",
                "tableName": "table2"
            }
        ],
        "targetField": {
            "dbName": "db_test",
            "fieldName": "name",
            "tableName": "table_result"
        }
    }
]
```

## 引用说明

```
<java.version>1.8</java.version>
<antlr4.version>4.7.2</antlr4.version>
// 通用的
import org.antlr.v4.runtime.ParserRuleContext;
import org.antlr.v4.runtime.RuleContext;
import org.antlr.v4.runtime.tree.ParseTree;
import org.apache.commons.lang3.StringUtils;
// 基于Hplsql.g4文件生成的, 使用antlr4-maven-plugin
import xxx.HplsqlBaseVisitor;
import xxx.HplsqlParser;
```