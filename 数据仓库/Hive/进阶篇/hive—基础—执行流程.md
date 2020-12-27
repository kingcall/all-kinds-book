## SQL的执行流程
- org.apache.hadoop.hive.ql.Driver类是查询的起点，run()方法会先后调用compile()和execute()两个函数来完成查询，所以一个command的查询分为compile和execute两个阶段

![image-20201206212805939](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:28:06-image-20201206212805939.png)

1. 如果是quit或者exit，退出
2. 以source开头的，读取外部文件并执行文件中的HiveQL
3. !开头的命令，执行操作系统命令（如!ls，列出当前目录的文件信息）
4. list，列出jar/file/archive
5. 其他命令，则生成调用相应的CommandProcessor处理，进入CliDriver.processLocalCmd()
### Compile

(1)利用antlr生成的HiveLexer.java和HiveParser.java类，将HiveQL转换成抽象语法树（AST）。

首先使用antlr工具将srcqlsrcjavaorgapachehadoophiveqlparsehive.g编译成以下几个文件：HiveParser.java, Hive.tokens, Hive__.g, HiveLexer.java

HiveLexer.java和HiveParser.java分别是词法和语法分析类文件，Hive__.g是HiveLexer.java对应的词法分析规范，Hive.tokens定义了词法分析后所有的token。

然后沿着“Driver.compile()->ParseDriver.parse(command, ctx)->HiveParserX.statement()->antlr中的API”这个调用关系把输入的HiveQL转化成ASTNode类型的语法树。HiveParserX是由antlr生成的HiveParser类的子类。
### execute

(2)利用对应的SemanticAnalyzer类，将AST树转换成Map-reduce task

a)         AST -> Operator DAG

b)        优化Operator DAG

c)         Oprator DAG -> Map-reduce task

首先接着上一步生成的语法树ASTNode， SemanticAnalyzerFactory会根据ASTNode的token类型生成不同的SemanticAnalyzer (所有这些SemanticAnalyzer都继承自BaseSemanticAnalyzer)

1)      ExplainSemanticAnalyzer

2)      LoadSemanticAnalyzer

3)      ExportSemanticAnalyzer

4)      DDLSemanticAnalyzer

5)      FunctionSemanticAnalyzer

6)      SemanticAnalyzer