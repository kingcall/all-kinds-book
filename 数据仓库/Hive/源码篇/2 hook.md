Hive作为SQL on Hadoop最稳定、应用最广泛的查询引擎被大家所熟知。但是由于基于MapReduce，查询执行速度太慢而逐步引入其他的近实时查询引擎如Presto等。值得关注的是Hive目前支持MapReduce、Tez和Spark三种执行引擎，同时Hive3也会支持联邦数据查询的功能。所以Hive还是有很大进步的空间的。

当然，诸如SparkSQL和Presto有着他们非常合适的应用场景，我们的底层也是会有多种查询引擎存在，以应对不同业务场景的数据查询服务。但是由于查询引擎过多也会导致用户使用体验不好，需要用户掌握多种查询引擎，而且要明确知道各个引擎的适用场景。而且多种SQL引擎各自提供服务会对数据仓库建设过程中的血缘管理、权限管理、资源利用都带来较大的困难。

之前对于底层平台的统一SQL服务有考虑过在上层提供一层接口封装，进行SQL校验、血缘管理、引擎推荐、查询分发等等，但是各个引擎之间的语法差异较大，想要实现兼容的SQL层有点不太现实。最近看了快手分享的[《SQL on Hadoop 在快手大数据平台的实践与优化》](https://www.infoq.cn/article/BN9cJjg1t-QSWE6fqkoR)，觉得有那么点意思。大家有兴趣的话可以看一看。

其实快手的实现核心逻辑是一样的，有一个统一的SQL入口，提供SQL校验，SQL存储、引擎推荐、查询分发进而实现血缘管理等。优秀的是它基于Hive完成了上述工作，将Hive作为统一的入口而不是重新包装一层。既利用了HiveServer2的架构，又做到了对于用户的感知最小。而实现这些功能的基础就是Hive Hooks，也就是本篇的重点。

Hook是一种在处理过程中拦截事件，消息或函数调用的机制。 **Hive hooks是绑定到了Hive内部的工作机制，无需重新编译Hive。所以Hive Hook提供了使用hive扩展和集成外部功能的能力。** 我们可以通过Hive Hooks在查询处理的各个步骤中运行/注入一些代码，帮助我们实现想要实现的功能。

根据钩子的类型，它可以在查询处理期间的不同点调用：

`Pre-semantic-analyzer hooks`：在Hive在查询字符串上运行语义分析器之前调用。

`Post-semantic-analyzer hooks`：在Hive在查询字符串上运行语义分析器之后调用。

`Pre-driver-run hooks`：在driver执行查询之前调用。

`Post-driver-run hooks`：在driver执行查询之后调用。

`Pre-execution hooks`：在执行引擎执行查询之前调用。请注意，这个目的是此时已经为Hive准备了一个优化的查询计划。

`Post-execution hooks`：在查询执行完成之后以及将结果返回给用户之前调用。

`Failure-execution hooks`：当查询执行失败时调用。

由以上的Hive Hooks我们都可以得出Hive SQL执行的生命周期了，而Hive Hooks则完整的贯穿了Hive查询的整个生命周期。

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/20210113174459.png)

对于Hive Hooks有了初步理解之后，后面我们会通过示例介绍如何实现一个Hive Hook，并且尝试一下如何基于Hive实现统一的SQL查询服务。

### SemanticAnalyzerHook 来过滤不加分区条件的Hive查询

  我们Hadoop集群中将近百分之80的作业是通过Hive来提交的，由于Hive写起来简单便捷，而且我们又提供了Hive Web Client，所以使用范围很广，包括ba，pm，po，sales都在使用hive进行ad-hoc查询，但是hive在降低用户使用门槛的同时，也使得用户经常写不合理开销很大的语句，生成了很多的mapreduce job，占用了大量slot数，其中最典型的例子就是分区表查询，不指定分区条件，导致hive没有做partition pruner优化，进而读入了所有的表数据，占用大量IO和计算资源。

具体做法是实现HiveSemanticAnalyzerHook接口，preAnalyze方法和postAnalyze方法会分别在compile函数之前和之后执行，我们只要实现preAnalyze方法，遍历传进来的ASTNode抽象语法树，获取左子树的From表名和右子树的where判断条件key值，如果该From表是分区表的话，会通过metastore client获取它的所有分区key名字，用户指定的where条件中只要出现任何一个分区key，则此语句通过检测，否则会在标准错误中输出一条warning，并且在后台log中记录用户名和执行语句，每隔一段时间会将这些bad case在hive-user组邮箱进行公示，希望能通过这种方式来起到相互警示和学习的效果.



```java
package org.apache.hadoop.hive.ql.parse;
 
import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;
 
import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.hive.metastore.api.FieldSchema;
import org.apache.hadoop.hive.ql.exec.Task;
import org.apache.hadoop.hive.ql.metadata.Hive;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.metadata.Table;
import org.apache.hadoop.hive.ql.session.SessionState;
import org.apache.hadoop.hive.ql.session.SessionState.LogHelper;
import org.apache.hadoop.hive.shims.ShimLoader;
 
public class DPSemanticAnalyzerHook extends AbstractSemanticAnalyzerHook {
  private final static String NO_PARTITION_WARNING = "WARNING: HQL is not efficient, Please specify partition condition! HQL:%s ;USERNAME:%s";
 
  private final SessionState ss = SessionState.get();
  private final LogHelper console = SessionState.getConsole();
  private Hive hive = null;
  private String username;
  private String currentDatabase = "default";
  private String hql;
  private String whereHql;
  private String tableAlias;
  private String tableName;
  private String tableDatabaseName;
  private Boolean needCheckPartition = false;
 
  @Override
  public ASTNode preAnalyze(HiveSemanticAnalyzerHookContext context, ASTNode ast)
      throws SemanticException {
    try {
      hql = ss.getCmd().toLowerCase();
      hql = StringUtils.replaceChars(hql, '\n', ' ');
      if (hql.contains("where")) {
        whereHql = hql.substring(hql.indexOf("where"));
      }
      username = ShimLoader.getHadoopShims().getUserName(context.getConf());
 
      if (ast.getToken().getType() == HiveParser.TOK_QUERY) {
        try {
          hive = context.getHive();
          currentDatabase = hive.getCurrentDatabase();
        } catch (HiveException e) {
          throw new SemanticException(e);
        }
 
        extractFromClause((ASTNode) ast.getChild(0));
 
        if (needCheckPartition && !StringUtils.isBlank(tableName)) {
          String dbname = StringUtils.isEmpty(tableDatabaseName) ? currentDatabase
              : tableDatabaseName;
          String tbname = tableName;
          String[] parts = tableName.split(".");
          if (parts.length == 2) {
            dbname = parts[0];
            tbname = parts[1];
          }
          Table t = hive.getTable(dbname, tbname);
          if (t.isPartitioned()) {
            if (StringUtils.isBlank(whereHql)) {
              console.printError(String.format(NO_PARTITION_WARNING, hql, username));
            } else {
              List<FieldSchema> partitionKeys = t.getPartitionKeys();
              List<String> partitionNames = new ArrayList<String>();
              for (int i = 0; i < partitionKeys.size(); i++) {
                partitionNames.add(partitionKeys.get(i).getName().toLowerCase());
              }
 
              if (!containsPartCond(partitionNames, whereHql, tableAlias)) {
                console.printError(String.format(NO_PARTITION_WARNING, hql, username));
              }
            }
          }
        }
 
      }
    } catch (Exception ex) {
      ex.printStackTrace();
    }
    return ast;
  }
 
  private boolean containsPartCond(List<String> partitionKeys, String sql, String alias) {
    for (String pk : partitionKeys) {
      if (sql.contains(pk)) {
        return true;
      }
      if (!StringUtils.isEmpty(alias) && sql.contains(alias + "." + pk)) {
        return true;
      }
    }
    return false;
  }
 
  private void extractFromClause(ASTNode ast) {
    if (HiveParser.TOK_FROM == ast.getToken().getType()) {
      ASTNode refNode = (ASTNode) ast.getChild(0);
      if (refNode.getToken().getType() == HiveParser.TOK_TABREF && ast.getChildCount() == 1) {
        ASTNode tabNameNode = (ASTNode) (refNode.getChild(0));
        int refNodeChildCount = refNode.getChildCount();
        if (tabNameNode.getToken().getType() == HiveParser.TOK_TABNAME) {
          if (tabNameNode.getChildCount() == 2) {
            tableDatabaseName = tabNameNode.getChild(0).getText().toLowerCase();
            tableName = BaseSemanticAnalyzer.getUnescapedName((ASTNode) tabNameNode.getChild(1))
                .toLowerCase();
          } else if (tabNameNode.getChildCount() == 1) {
            tableName = BaseSemanticAnalyzer.getUnescapedName((ASTNode) tabNameNode.getChild(0))
                .toLowerCase();
          } else {
            return;
          }
 
          if (refNodeChildCount == 2) {
            tableAlias = BaseSemanticAnalyzer.unescapeIdentifier(refNode.getChild(1).getText())
                .toLowerCase();
          }
          needCheckPartition = true;
        }
      }
    }
  }
 
  @Override
  public void postAnalyze(HiveSemanticAnalyzerHookContext context,
      List<Task<? extends Serializable>> rootTasks) throws SemanticException {
    // LogHelper console = SessionState.getConsole();
    // Set<ReadEntity> readEntitys = context.getInputs();
    // console.printInfo("Total Read Entity Size:" + readEntitys.size());
    // for (ReadEntity readEntity : readEntitys) {
    // Partition p = readEntity.getPartition();
    // Table t = readEntity.getTable();
    // }
  }
}
```

