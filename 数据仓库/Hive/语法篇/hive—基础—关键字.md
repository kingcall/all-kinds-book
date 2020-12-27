hive中有很多关键字，直接作为表名或列名，会出错的。

这里由张表`people_access_log`：

```
1	15110101010	1577003281739	112.168.1.2	https://www.baidu.com
2	15110101011	1577003281749	112.16.1.23	https://www.baidu.com
3	15110101012	1577003281759	193.168.1.2	https://www.taobao.com
4	15110101013	1577003281769	112.18.1.2	https://www.baidu.com
5	15110101014	1577003281779	112.168.10.2	https://www.baidu.com
6	15110101015	1577003281789	11.168.1.2	https://www.taobao.com
7	15110101016	1577003281790	112.168.1.3	https://www.qq.com
8	15110101017	1577003281791	112.1.1.3	https://www.microsoft.com
```

### 关键字直接作为表的别名

这里使用`user`这个关键字做表的别名，会报错：

```sql
select * from people_access_log user;
```

错误信息：

```
cannot recognize input near 'user' '<EOF>' '<EOF>' in table source
```

换一个表的别名，比如说`user1`就可以了，这种错误可能会在子查询中用户表使用`user`做别名，就会报错。

### 关键字直接作为列的别名

这里使用`user`这个关键字做列的别名，会报错：

```sql
select user_id user from people_access_log;
```

异常信息：

```
Failed to recognize predicate 'user'. Failed rule: 'identifier' in selection target
```

### 如何使用关键字做别名

如果想强行使用关键字也不是不可以，使用键盘Esc键下面那个点，包住别名即可，像下面这样，这在`sql`里面是通用的。

```sql
select user_id `user` from people_access_log;
```



## Hive中的关键字

在官网里面找到`Hive`中的关键字有下面这些，平时最好不要使用它们作为查询的别名。

| Version    | Non-reserved Keywords                                        | Reserved Keywords                                            |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Hive-1.2.0 | ADD, ADMIN, AFTER, ANALYZE, ARCHIVE, ASC, BEFORE, BUCKET, BUCKETS, CASCADE, CHANGE, CLUSTER, CLUSTERED, CLUSTERSTATUS, COLLECTION, COLUMNS, COMMENT, COMPACT, COMPACTIONS, COMPUTE, CONCATENATE, CONTINUE, DATA, DATABASES, DATETIME, DAY, DBPROPERTIES, DEFERRED, DEFINED, DELIMITED, DEPENDENCY, DESC, DIRECTORIES, DIRECTORY, DISABLE, DISTRIBUTE, ELEM_TYPE, ENABLE, ESCAPED, EXCLUSIVE, EXPLAIN, EXPORT, FIELDS, FILE, FILEFORMAT, FIRST, FORMAT, FORMATTED, FUNCTIONS, HOLD_DDLTIME, HOUR, IDXPROPERTIES, IGNORE, INDEX, INDEXES, INPATH, INPUTDRIVER, INPUTFORMAT, ITEMS, JAR, KEYS, KEY_TYPE, LIMIT, LINES, LOAD, LOCATION, LOCK, LOCKS, LOGICAL, LONG, MAPJOIN, MATERIALIZED, METADATA, MINUS, MINUTE, MONTH, MSCK, NOSCAN, NO_DROP, OFFLINE, OPTION, OUTPUTDRIVER, OUTPUTFORMAT, OVERWRITE, OWNER, PARTITIONED, PARTITIONS, PLUS, PRETTY, PRINCIPALS, PROTECTION, PURGE, READ, READONLY, REBUILD, RECORDREADER, RECORDWRITER, REGEXP, RELOAD, RENAME, REPAIR, REPLACE, REPLICATION, RESTRICT, REWRITE, RLIKE, ROLE, ROLES, SCHEMA, SCHEMAS, SECOND, SEMI, SERDE, SERDEPROPERTIES, SERVER, SETS, SHARED, SHOW, SHOW_DATABASE, SKEWED, SORT, SORTED, SSL, STATISTICS, STORED, STREAMTABLE, STRING, STRUCT, TABLES, TBLPROPERTIES, TEMPORARY, TERMINATED, TINYINT, TOUCH, TRANSACTIONS, UNARCHIVE, UNDO, UNIONTYPE, UNLOCK, UNSET, UNSIGNED, URI, USE, UTC, UTCTIMESTAMP, VALUE_TYPE, VIEW, WHILE, YEAR | ALL, ALTER, AND, ARRAY, AS, AUTHORIZATION, BETWEEN, BIGINT, BINARY, BOOLEAN, BOTH, BY, CASE, CAST, CHAR, COLUMN, CONF, CREATE, CROSS, CUBE, CURRENT, CURRENT_DATE, CURRENT_TIMESTAMP, CURSOR, DATABASE, DATE, DECIMAL, DELETE, DESCRIBE, DISTINCT, DOUBLE, DROP, ELSE, END, EXCHANGE, EXISTS, EXTENDED, EXTERNAL, FALSE, FETCH, FLOAT, FOLLOWING, FOR, FROM, FULL, FUNCTION, GRANT, GROUP, GROUPING, HAVING, IF, IMPORT, IN, INNER, INSERT, INT, INTERSECT, INTERVAL, INTO, IS, JOIN, LATERAL, LEFT, LESS, LIKE, LOCAL, MACRO, MAP, MORE, NONE, NOT, NULL, OF, ON, OR, ORDER, OUT, OUTER, OVER, PARTIALSCAN, PARTITION, PERCENT, PRECEDING, PRESERVE, PROCEDURE, RANGE, READS, REDUCE, REVOKE, RIGHT, ROLLUP, ROW, ROWS, SELECT, SET, SMALLINT, TABLE, TABLESAMPLE, THEN, TIMESTAMP, TO, TRANSFORM, TRIGGER, TRUE, TRUNCATE, UNBOUNDED, UNION, UNIQUEJOIN, UPDATE, USER, USING, UTC_TMESTAMP, VALUES, VARCHAR, WHEN, WHERE, WINDOW, WITH |
| Hive-2.0.0 | removed: REGEXP, RLIKE<br>added: AUTOCOMMIT, ISOLATION, LEVEL, OFFSET, SNAPSHOT, TRANSACTION, WORK, WRITE | added: COMMIT, ONLY, REGEXP, RLIKE, ROLLBACK, START          |
| Hive-2.1.0 | added: ABORT, KEY, LAST, NORELY, NOVALIDATE, NULLS, RELY, VALIDATE | added: CACHE, CONSTRAINT, FOREIGN, PRIMARY, REFERENCES       |
| Hive-2.2.0 | added: DETAIL, DOW, EXPRESSION, OPERATOR, QUARTER, SUMMARY, VECTORIZATION, WEEK, YEARS, MONTHS, WEEKS, DAYS, HOURS, MINUTES, SECONDS | added: DAYOFWEEK, EXTRACT, FLOOR, INTEGER, PRECISION, VIEWS  |
| Hive 3.0.0 | added: TIMESTAMPTZ, ZONE                                     | added: TIME, NUMERIC, SYNC                                   |


Reserved keywords are permitted as identifiers if you quote them as described in Supporting Quoted Identifiers in Column Names (version 0.13.0 and later, see HIVE-6013). Most of the keywords are reserved through HIVE-6617 in order to reduce the ambiguity in grammar (version 1.2.0 and later). There are two ways if the user still would like to use those reserved keywords as identifiers: (1) use quoted identifiers, (2) set hive.support.sql11.reserved.keywords=false. (version 2.1.0 and earlier) 