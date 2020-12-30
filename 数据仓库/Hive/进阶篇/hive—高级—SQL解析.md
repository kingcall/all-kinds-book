[TOC]

## Antlr

- Hive使用Antlr实现SQL的词法和语法解析。
- Antlr是一种语言识别的工具，可以用来构造领域语言。
这里不详细介绍Antlr，只需要了解使用Antlr构造特定的语言只需要编写一个语法文件，定义词法和语法替换规则即可，Antlr完成了词法分析、语法分析、语义分析、中间代码生成的过程。
- Hive 中语法规则的定义文件在0.10版本以前是Hive.g一个文件，随着语法规则越来越复杂，由语法规则生成的Java解析类可能超过Java类文件的最大 上限，0.11版本将Hive.g拆成了5个文件，词法规则HiveLexer.g和语法规则的4个文件 SelectClauseParser.g，FromClauseParser.g，IdentifiersParser.g，HiveParser.g。

### sql 解析生成语法树
- Antlr对Hive SQL解析的代码如下，HiveLexerX，HiveParser分别是Antlr对语法文件Hive.g编译后自动生成的词法解析和语法解析类，在这两个类中进行复杂的解析。

```
HiveLexerX lexer = new HiveLexerX(new ANTLRNoCaseStringStream(command));    //词法解析，忽略关键词的大小写
TokenRewriteStream tokens = new TokenRewriteStream(lexer);
if (ctx != null) {
  ctx.setTokenRewriteStream(tokens);
}
HiveParser parser = new HiveParser(tokens);                                 //语法解析
parser.setTreeAdaptor(adaptor);
HiveParser.statement_return r = null;
try {
  r = parser.statement();                                                   //转化为AST Tree
} catch (RecognitionException e) {
  e.printStackTrace();
  throw new ParseException(parser.errors);
}
```

### SQL基本组成单元QueryBlock
- AST Tree仍然非常复杂，不够结构化，不方便直接翻译为MapReduce程序，AST Tree转化为QueryBlock就是将SQL进一部抽象和结构化。
- QueryBlock是一条SQL最基本的组成单元，包括三个部分：输入源，计算过程，输出。简单来讲一个QueryBlock就是一个子查询。
- AST Tree生成QueryBlock的过程是一个递归的过程，先序遍历AST Tree，遇到不同的Token节点，保存到相应的属性中，主要包含以下几个过程

### QueryBlock生成Operator Tree
- QueryBlock生成Operator Tree就是遍历上一个过程中生成的QB和QBParseInfo对象的保存语法的属性

### 逻辑层优化器

### OperatorTree生成MapReduce Job的过程





## 解析

Hive 中的 hive-exec 模块是包含 SQL 解析模块的. 因此项目的 pom.xml 中加上.

```
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>${hive.version}</version>
    <classifier>core</classifier>
</dependency>
```

需要注意的是, `<classifier>core</classifier>` 必不可少, 因为 2.1.1 版本的 hive-exec 默认打包是将所有依赖塞进一个 fat jar 中, 一个 jar 35 MB 大小, 我们仅仅想拆个 SQL, 用不了这么多