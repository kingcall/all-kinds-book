hive中有很多关键字，直接作为`列名或者表名`，会出错的，例如`user`这个关键字很多人会拿来做表的别名，就会出错：

```sql
hive> select * from user_log user;

NoViableAltException(311@[157:5: ( ( Identifier LPAREN )=> partitionedTableFunction | tableSource | subQuerySource | virtualTableSource )])
	at org.antlr.runtime.DFA.noViableAlt(DFA.java:158)
	at org.antlr.runtime.DFA.predict(DFA.java:116)
	at org.apache.hadoop.hive.ql.parse.HiveParser_FromClauseParser.fromSource0(HiveParser_FromClauseParser.java:2901)
	at org.apache.hadoop.hive.ql.parse.HiveParser_FromClauseParser.fromSource(HiveParser_FromClauseParser.java:2839)
	at org.apache.hadoop.hive.ql.parse.HiveParser_FromClauseParser.joinSource(HiveParser_FromClauseParser.java:1410)
	at org.apache.hadoop.hive.ql.parse.HiveParser_FromClauseParser.fromClause(HiveParser_FromClauseParser.java:1300)
	at org.apache.hadoop.hive.ql.parse.HiveParser.fromClause(HiveParser.java:39647)
	at org.apache.hadoop.hive.ql.parse.HiveParser.selectStatement(HiveParser.java:34897)
	at org.apache.hadoop.hive.ql.parse.HiveParser.regularBody(HiveParser.java:34803)
	at org.apache.hadoop.hive.ql.parse.HiveParser.queryStatementExpressionBody(HiveParser.java:33992)
	at org.apache.hadoop.hive.ql.parse.HiveParser.queryStatementExpression(HiveParser.java:33880)
	at org.apache.hadoop.hive.ql.parse.HiveParser.execStatement(HiveParser.java:2205)
	at org.apache.hadoop.hive.ql.parse.HiveParser.statement(HiveParser.java:1313)
	at org.apache.hadoop.hive.ql.parse.ParseDriver.parse(ParseDriver.java:218)
	at org.apache.hadoop.hive.ql.parse.ParseUtils.parse(ParseUtils.java:75)
	at org.apache.hadoop.hive.ql.parse.ParseUtils.parse(ParseUtils.java:68)
	at org.apache.hadoop.hive.ql.Driver.compile(Driver.java:525)
	at org.apache.hadoop.hive.ql.Driver.compileInternal(Driver.java:1359)
	at org.apache.hadoop.hive.ql.Driver.runInternal(Driver.java:1488)
	at org.apache.hadoop.hive.ql.Driver.run(Driver.java:1278)
	at org.apache.hadoop.hive.ql.Driver.run(Driver.java:1268)
	at org.apache.hadoop.hive.cli.CliDriver.processLocalCmd(CliDriver.java:239)
	at org.apache.hadoop.hive.cli.CliDriver.processCmd(CliDriver.java:187)
	at org.apache.hadoop.hive.cli.CliDriver.processLine(CliDriver.java:409)
	at org.apache.hadoop.hive.cli.CliDriver.executeDriver(CliDriver.java:838)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:774)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:701)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.util.RunJar.run(RunJar.java:313)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:227)
FAILED: ParseException line 1:23 cannot recognize input near 'user_log' 'user' '<EOF>' in from source 0
```

`user`直接做字段的别名，也是会出错的

```sql
hive> select userid user from user_log ;
NoViableAltException(311@[123:1: selectItem : ( ( tableAllColumns )=> tableAllColumns -> ^( TOK_SELEXPR tableAllColumns ) | ( expression ( ( ( KW_AS )? identifier ) | ( KW_AS LPAREN identifier ( COMMA identifier )* RPAREN ) )? ) -> ^( TOK_SELEXPR expression ( identifier )* ) );])
	at org.antlr.runtime.DFA.noViableAlt(DFA.java:158)
	at org.antlr.runtime.DFA.predict(DFA.java:116)
	at org.apache.hadoop.hive.ql.parse.HiveParser_SelectClauseParser.selectItem(HiveParser_SelectClauseParser.java:2244)
	at org.apache.hadoop.hive.ql.parse.HiveParser_SelectClauseParser.selectList(HiveParser_SelectClauseParser.java:1144)
	at org.apache.hadoop.hive.ql.parse.HiveParser_SelectClauseParser.selectClause(HiveParser_SelectClauseParser.java:939)
	at org.apache.hadoop.hive.ql.parse.HiveParser.selectClause(HiveParser.java:39575)
	at org.apache.hadoop.hive.ql.parse.HiveParser.selectStatement(HiveParser.java:34882)
	at org.apache.hadoop.hive.ql.parse.HiveParser.regularBody(HiveParser.java:34803)
	at org.apache.hadoop.hive.ql.parse.HiveParser.queryStatementExpressionBody(HiveParser.java:33992)
	at org.apache.hadoop.hive.ql.parse.HiveParser.queryStatementExpression(HiveParser.java:33880)
	at org.apache.hadoop.hive.ql.parse.HiveParser.execStatement(HiveParser.java:2205)
	at org.apache.hadoop.hive.ql.parse.HiveParser.statement(HiveParser.java:1313)
	at org.apache.hadoop.hive.ql.parse.ParseDriver.parse(ParseDriver.java:218)
	at org.apache.hadoop.hive.ql.parse.ParseUtils.parse(ParseUtils.java:75)
	at org.apache.hadoop.hive.ql.parse.ParseUtils.parse(ParseUtils.java:68)
	at org.apache.hadoop.hive.ql.Driver.compile(Driver.java:525)
	at org.apache.hadoop.hive.ql.Driver.compileInternal(Driver.java:1359)
	at org.apache.hadoop.hive.ql.Driver.runInternal(Driver.java:1488)
	at org.apache.hadoop.hive.ql.Driver.run(Driver.java:1278)
	at org.apache.hadoop.hive.ql.Driver.run(Driver.java:1268)
	at org.apache.hadoop.hive.cli.CliDriver.processLocalCmd(CliDriver.java:239)
	at org.apache.hadoop.hive.cli.CliDriver.processCmd(CliDriver.java:187)
	at org.apache.hadoop.hive.cli.CliDriver.processLine(CliDriver.java:409)
	at org.apache.hadoop.hive.cli.CliDriver.executeDriver(CliDriver.java:838)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:774)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:701)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.util.RunJar.run(RunJar.java:313)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:227)
FAILED: ParseException line 1:14 cannot recognize input near 'userid' 'user' 'from' in selection target
```

如果实在想用关键字做别名的话，可以用键盘上面`Esc`下面那个“点”包裹起来，像这样：

```sql
-- user关键字做表的别名
select * from user_log `user`;

-- user关键字做列的别名
select userid `user` from user_log ;

```



##　Ｈive中的关键字

| Version    | Non-reserved Keywords                                        | Reserved Keywords                                            |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Hive-1.2.0 | ADD, ADMIN, AFTER, ANALYZE, ARCHIVE, ASC, BEFORE, BUCKET, BUCKETS, CASCADE, CHANGE, CLUSTER, CLUSTERED, CLUSTERSTATUS, COLLECTION, COLUMNS, COMMENT, COMPACT, COMPACTIONS, COMPUTE, CONCATENATE, CONTINUE, DATA, DATABASES, DATETIME, DAY, DBPROPERTIES, DEFERRED, DEFINED, DELIMITED, DEPENDENCY, DESC, DIRECTORIES, DIRECTORY, DISABLE, DISTRIBUTE, ELEM_TYPE, ENABLE, ESCAPED, EXCLUSIVE, EXPLAIN, EXPORT, FIELDS, FILE, FILEFORMAT, FIRST, FORMAT, FORMATTED, FUNCTIONS, HOLD_DDLTIME, HOUR, IDXPROPERTIES, IGNORE, INDEX, INDEXES, INPATH, INPUTDRIVER, INPUTFORMAT, ITEMS, JAR, KEYS, KEY_TYPE, LIMIT, LINES, LOAD, LOCATION, LOCK, LOCKS, LOGICAL, LONG, MAPJOIN, MATERIALIZED, METADATA, MINUS, MINUTE, MONTH, MSCK, NOSCAN, NO_DROP, OFFLINE, OPTION, OUTPUTDRIVER, OUTPUTFORMAT, OVERWRITE, OWNER, PARTITIONED, PARTITIONS, PLUS, PRETTY, PRINCIPALS, PROTECTION, PURGE, READ, READONLY, REBUILD, RECORDREADER, RECORDWRITER, REGEXP, RELOAD, RENAME, REPAIR, REPLACE, REPLICATION, RESTRICT, REWRITE, RLIKE, ROLE, ROLES, SCHEMA, SCHEMAS, SECOND, SEMI, SERDE, SERDEPROPERTIES, SERVER, SETS, SHARED, SHOW, SHOW_DATABASE, SKEWED, SORT, SORTED, SSL, STATISTICS, STORED, STREAMTABLE, STRING, STRUCT, TABLES, TBLPROPERTIES, TEMPORARY, TERMINATED, TINYINT, TOUCH, TRANSACTIONS, UNARCHIVE, UNDO, UNIONTYPE, UNLOCK, UNSET, UNSIGNED, URI, USE, UTC, UTCTIMESTAMP, VALUE_TYPE, VIEW, WHILE, YEAR | ALL, ALTER, AND, ARRAY, AS, AUTHORIZATION, BETWEEN, BIGINT, BINARY, BOOLEAN, BOTH, BY, CASE, CAST, CHAR, COLUMN, CONF, CREATE, CROSS, CUBE, CURRENT, CURRENT_DATE, CURRENT_TIMESTAMP, CURSOR, DATABASE, DATE, DECIMAL, DELETE, DESCRIBE, DISTINCT, DOUBLE, DROP, ELSE, END, EXCHANGE, EXISTS, EXTENDED, EXTERNAL, FALSE, FETCH, FLOAT, FOLLOWING, FOR, FROM, FULL, FUNCTION, GRANT, GROUP, GROUPING, HAVING, IF, IMPORT, IN, INNER, INSERT, INT, INTERSECT, INTERVAL, INTO, IS, JOIN, LATERAL, LEFT, LESS, LIKE, LOCAL, MACRO, MAP, MORE, NONE, NOT, NULL, OF, ON, OR, ORDER, OUT, OUTER, OVER, PARTIALSCAN, PARTITION, PERCENT, PRECEDING, PRESERVE, PROCEDURE, RANGE, READS, REDUCE, REVOKE, RIGHT, ROLLUP, ROW, ROWS, SELECT, SET, SMALLINT, TABLE, TABLESAMPLE, THEN, TIMESTAMP, TO, TRANSFORM, TRIGGER, TRUE, TRUNCATE, UNBOUNDED, UNION, UNIQUEJOIN, UPDATE, USER, USING, UTC_TMESTAMP, VALUES, VARCHAR, WHEN, WHERE, WINDOW, WITH |
| Hive-2.0.0 | removed: REGEXP, RLIKE<br>added: AUTOCOMMIT, ISOLATION, LEVEL, OFFSET, SNAPSHOT, TRANSACTION, WORK, WRITE | added: COMMIT, ONLY, REGEXP, RLIKE, ROLLBACK, START          |
| Hive-2.1.0 | added: ABORT, KEY, LAST, NORELY, NOVALIDATE, NULLS, RELY, VALIDATE | added: CACHE, CONSTRAINT, FOREIGN, PRIMARY, REFERENCES       |
| Hive-2.2.0 | added: DETAIL, DOW, EXPRESSION, OPERATOR, QUARTER, SUMMARY, VECTORIZATION, WEEK, YEARS, MONTHS, WEEKS, DAYS, HOURS, MINUTES, SECONDS | added: DAYOFWEEK, EXTRACT, FLOOR, INTEGER, PRECISION, VIEWS  |
| Hive 3.0.0 | added: TIMESTAMPTZ, ZONE                                     | added: TIME, NUMERIC, SYNC                                   |


Reserved keywords are permitted as identifiers if you quote them as described in Supporting Quoted Identifiers in Column Names (version 0.13.0 and later, see HIVE-6013). Most of the keywords are reserved through HIVE-6617 in order to reduce the ambiguity in grammar (version 1.2.0 and later). There are two ways if the user still would like to use those reserved keywords as identifiers: (1) use quoted identifiers, (2) set hive.support.sql11.reserved.keywords=false. (version 2.1.0 and earlier) 