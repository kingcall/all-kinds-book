[toc]
- MySQL 服务器可以在不同的 SQL 模式下运行，并且可以针对不同的客户端以不同的方式应用这些模式，具体取决于 sql_mode 系统变量的值。DBA 可以设置全局SQL模式以匹配站点服务器操作要求，并且每个应用程序可以将其会话 SQL 模式设置为其自己的要求。
- 模式会影响 **MySQL 支持的 SQL语法以及它执行的 数据验证检查**，这使得在不同环境中使用MySQL以及将MySQL与其他数据库服务器一起使用变得更加容易。

## 默认模式
- 当我们没有修改配置文件的情况下，MySQL 是有自己的默认模式的；版本不同，默认模式也不同

```
-- 查看 MySQL 版本
SELECT VERSION();

-- 查看 sql_mode
SELECT @@sql_mode;
```

## 模式
### 语法支持类　　

####  ONLY_FULL_GROUP_BY
- 对于 GROUP BY 聚合操作，如果在 SELECT 中的列、HAVING 或者 ORDER BY 子句的列，没有在GROUP BY中出现，那么这个SQL是不合法的

#### ANSI_QUOTES
- 启用 ANSI_QUOTES 后，不能用双引号来引用字符串，因为它被解释为识别符，作用与 ` 一样。设置它以后，update t set f1="" …，会报 Unknown column ‘’ in field list 这样的语法错误

#### PIPES_AS_CONCAT
- 将 || 视为字符串的连接操作符而非 或 运算符，这和Oracle数据库是一样的，也和字符串的拼接函数 CONCAT() 相类似

#### NO_TABLE_OPTIONS
- 使用 SHOW CREATE TABLE 时不会输出MySQL特有的语法部分，如 ENGINE ，这个在使用 mysqldump 跨DB种类迁移的时候需要考虑

#### NO_AUTO_CREATE_USER
- 字面意思不自动创建用户。在给MySQL用户授权时，我们习惯使用 GRANT … ON … TO dbuser 顺道一起创建用户。设置该选项后就与oracle操作类似，授权之前必须先建立用户

### 数据检查类　　
#### NO_ZERO_DATE

认为日期 ‘0000-00-00’ 非法，与是否设置后面的严格模式有关

1、如果设置了严格模式，则 NO_ZERO_DATE 自然满足。但如果是 INSERT IGNORE 或 UPDATE IGNORE，’0000-00-00’依然允许且只显示warning；

2、如果在非严格模式下，设置了NO_ZERO_DATE，效果与上面一样，’0000-00-00’ 允许但显示warning；如果没有设置NO_ZERO_DATE，no warning，当做完全合法的值；

3、NO_ZERO_IN_DATE情况与上面类似，不同的是控制日期和天，是否可为 0 ，即 2010-01-00 是否合法；

#### NO_ENGINE_SUBSTITUTION

使用 ALTER TABLE 或 CREATE TABLE 指定 ENGINE 时， 需要的存储引擎被禁用或未编译，该如何处理。启用 NO_ENGINE_SUBSTITUTION 时，那么直接抛出错误；不设置此值时，CREATE用默认的存储引擎替代，ATLER不进行更改，并抛出一个 warning

#### STRICT_TRANS_TABLES

设置它，表示启用严格模式。注意 STRICT_TRANS_TABLES 不是几种策略的组合，单独指 INSERT、UPDATE 出现少值或无效值该如何处理：

1、前面提到的把 ‘’ 传给int，严格模式下非法，若启用非严格模式则变成 0，产生一个warning；

2、Out Of Range，变成插入最大边界值；

3、当要插入的新行中，不包含其定义中没有显式DEFAULT子句的非NULL列的值时，该列缺少值；
