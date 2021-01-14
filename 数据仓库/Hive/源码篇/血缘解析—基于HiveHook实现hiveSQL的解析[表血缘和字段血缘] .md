[TOC]

## 前言

hive在中间执行过程中留下不少钩子可以供开发者开发拓展功能，大致有如下几个

- driver run的时候
- 执行计划semanticAnalyze前后
- 查询放入job之前
- exec前后
- 执行失败时

下面引用一份完整的hive中hook的流程，包括相应的配置项。

```
Driver.run()

=> HiveDriverRunHook.preDriverRun()(hive.exec.driver.run.hooks)

=> Driver.compile()

=> HiveSemanticAnalyzerHook.preAnalyze()(hive.semantic.analyzer.hook)

=> SemanticAnalyze(QueryBlock, LogicalPlan, PhyPlan, TaskTree)

=> HiveSemanticAnalyzerHook.postAnalyze()(hive.semantic.analyzer.hook)

=> QueryString redactor(hive.exec.query.redactor.hooks)

=> QueryPlan Generation

=> Authorization

=> Driver.execute()

=> ExecuteWithHookContext.run() || PreExecute.run() (hive.exec.pre.hooks)

=> TaskRunner

=> if failed, ExecuteWithHookContext.run()(hive.exec.failure.hooks)

=> ExecuteWithHookContext.run() || PostExecute.run() (hive.exec.post.hooks)

=> HiveDriverRunHook.postDriverRun()(hive.exec.driver.run.hooks)
```

## 血缘解析

这边要举的具体例子为利用hive自带的血缘解析写一个hook。

首先确认hook主要要实现的是ExecuteWithHookContext的run方法，且它带一个参数HookContext，包含了几乎所有的信息

因为是要取的是血缘，得拿到的是正确执行的那部分，所以hook放在执行后的hive.exec.post.hooks，可以避免执行失败等问题。

新建一个maven项目,引用hive-exec，版本按照hive的来

```
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>1.2.1</version>
</dependency>
```

直接上代码，表和字段级直接都放里面了

```java
package cn.ganjiacheng;

import org.apache.hadoop.hive.ql.hooks.ExecuteWithHookContext;
import org.apache.hadoop.hive.ql.hooks.HookContext;
import org.apache.hadoop.hive.ql.hooks.LineageInfo;
import org.apache.hadoop.hive.metastore.api.Table;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.*;

/**
 * @description:
 * @author: again
 * @email: ganjiacheng@souche.com
 * @date: 2020/5/4 3:18 下午
 */
public class MyLineagehook implements ExecuteWithHookContext {

    private Logger logger = LoggerFactory.getLogger(MyLineagehook.class);

    // 输出表
    private Set<String> inputTables;

    // 输入表
    private Set<String> outputTables;

    // 字段血缘 Map 
    // key为输出字段，value为来源字段数组
    private Map<String, ArrayList<String>> fieldLineage;

    public MyLineagehook() {
        inputTables = new HashSet<>();
        outputTables = new HashSet<>();
        fieldLineage = new HashMap<>();
    }

    // 处理表的格式为 库.表
    private String dealOutputTable(Table table) {
        String dbName = table.getDbName();
        String tableName = table.getTableName();
        return dbName != null ? String.format("%s.%s", dbName, tableName) : tableName;
    }

    // 处理输出字段的格式
    private String dealDepOutputField(LineageInfo.DependencyKey dependencyKey) {
        try{
            String tableName = dealOutputTable(dependencyKey.getDataContainer().getTable());
            String field = dependencyKey.getFieldSchema().getName();
            return String.format("%s.%s", tableName, field);
        }catch (Exception e) {
            logger.error("deal dep output field error" + e.getMessage());
            return null;
        }
    }

    // 处理来源字段的格式
    private String dealBaseOutputField(LineageInfo.BaseColumnInfo baseColumnInfo) {
        try{
            String tableName = dealOutputTable(baseColumnInfo.getTabAlias().getTable());
            String field = baseColumnInfo.getColumn().getName();
            return String.format("%s.%s", tableName, field);
        }catch (Exception e) {
            logger.error("deal base output field error" + e.getMessage());
            return null;
        }
    }

    // 主要重写的方法，入口，
    // 血缘的信息在hookContext.getLinfo()
    // 结构是集合，每个是一个map，代表一个字段的血缘，
    // 每个map的key为输出字段，value为来源字段
    // 处理表血缘就直接忽略字段，因为存在set里就避免重复
    // 处理字段血缘就直接分别处理key value的每个即可，最终也存储在类似的map中
    @Override
    public void run(HookContext hookContext) {
        for(Map.Entry<LineageInfo.DependencyKey, LineageInfo.Dependency> dep: hookContext.getLinfo().entrySet()){
            // 表血缘
            Optional.ofNullable(dep.getKey())
                    .map(LineageInfo.DependencyKey::getDataContainer)
                    .map(LineageInfo.DataContainer::getTable)
                    .map(this::dealOutputTable)
                    .ifPresent(outputTables::add);
            Optional.ofNullable(dep.getValue())
                    .map(LineageInfo.Dependency::getBaseCols)
                    .ifPresent(items -> items.stream().map(LineageInfo.BaseColumnInfo::getTabAlias)
                            .map(LineageInfo.TableAliasInfo::getTable)
                            .map(this::dealOutputTable)
                            .forEach(inputTables::add));

            // 字段血缘
            String column = Optional.ofNullable(dep.getKey())
                    .map(this::dealDepOutputField)
                    .map(aimField -> {
                        fieldLineage.put(aimField, new ArrayList<>());
                        return aimField;
                    }).orElse(null);
            Optional.ofNullable(dep.getValue())
                    .map(LineageInfo.Dependency::getBaseCols)
                    .ifPresent(items -> items.stream()
                            .map(this::dealBaseOutputField)
                            .forEach(item -> {
                                fieldLineage.get(column).add(item);
                            }));
        }
        System.out.println("来源表:");
        System.out.println(inputTables);
        System.out.println("输出表:");
        System.out.println(outputTables);
        System.out.println("字段血缘:");
        System.out.println(fieldLineage.toString());
    }

}
```

## 使用方法

编译后生成jar文件

```
启动hive
>hive

添加jar包
>add jar xxx.jar;

设置hook
>set hive.exec.post.hooks=cn.ganjiacheng.MyLineagehook;

运行一个insert语句
```

效果展示

```
INSERT OVERWRITE TABLE myuser_info
SELECT
    m.id AS id,
    m.name AS name,
    mp.phone AS phone,
    me.email AS email
FROM
    myuser m
LEFT JOIN
(
    SELECT
        *
    FROM
        myuser_phone
) mp
ON m.id=mp.id
LEFT JOIN
    myuser_email me
ON m.id=me.id
来源表:
[default.myuser_phone, default.myuser_email, default.myuser]
输出表:
[default.myuser_info]
字段血缘:
{default.myuser_info.phone=[default.myuser_phone.phone], default.myuser_info.email=[default.myuser_email.email], default.myuser_info.id=[default.myuser.id], default.myuser_info.name=[default.myuser.name]}
```

## 总结

这边的实现比直接用antlr4解析方便很多，代码量也比较少。

这边比直接解析sql好的一点是之前use的库会自动帮你补全到字段血缘中，但直接解析sql就无法知道库。

还有这边也是直接支持 select * 这种表达式的元数据获取，但光解析sql就无法和元数据连接。

这边的hivehook解析完数据后，可以通过消息发送到MQ中，后续后端进行采集消费，这边不做拓展。