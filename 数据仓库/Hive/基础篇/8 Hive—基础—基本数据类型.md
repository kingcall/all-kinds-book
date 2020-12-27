[TOC]

## 数据类型

在创建表的时候我们**在新建列的时候会需要指定数据类型**，一般在数据文件中我们可以将所有的数据都指定为string即字符串类型，然后再用函数转换为其他的数据类型，比如日期、数值等。

```
CREATE  TABLE [IF NOT EXISTS] [db_name.]table_name    
  ``[(col_name data_type [COMMENT col_comment], ... [constraint_specification])]
  data_type
  : primitive_type
  | array_type
  | map_type
  | struct_type
  | union_type
```

由hive的语法规则可以看到hive的数据类型大致可以分为5种，但是这5种又大致可以分为2类，第一类就是我们说的**原始数据类型**，或者说是**基本数据类型**，另外一类我们统称为复杂数据结构，我们下一节学习

### 数值类型

| 类型     | 占用字节数 | 存储范围                                                    | 默认类型    | 例子 | 备注                                          |
| :------- | :--------- | ----------------------------------------------------------- | ----------- | ---- | :-------------------------------------------- |
| TINYINT  | 1字节      | `-128` to `127`                                             | INT 类型    | 0    | 有符号整型                                    |
| SMALLINT | 2字节      | `-32,768` to `32,767`                                       | INT 类型    | 0    | 有符号整型                                    |
| INT      | 4字节      | `-2,147,483,648` to `2,147,483,647`                         | INT 类型    | 0    | 有符号整型                                    |
| BIGINT   | 8字节      | `-9,223,372,036,854,775,808` to `9,223,372,036,854,775,807` | INT 类型    | 0    | 有符号整型                                    |
| FLOAT    | 4字节      |                                                             | DOUBLE 类型 |      | 有符号单精度浮点数                            |
| DOUBLE   | 8字节      |                                                             | DOUBLE 类型 |      | 有符号双精度浮点数                            |
| DECIMAL  | --         |                                                             |             |      | 可带小数的精确数字字符串  Hive 0.11.0版本可用 |
| NUMERIC  |            |                                                             |             |      | 和DECIMAL一样，Hive 3.0.0版本可用             |

需要注意的是所有的这些数据类型都是对Java中的接口的实现，因此这些类型的具体行为细节和Java中对应的类型是完全一致的。例如，STRING类型实现的是Java中的String，FLOAT实现的是Java中的float，等等

#### 整型

其中整型类型包括(`TINYINT`, `SMALLINT`, `INT/INTEGER`, `BIGINT`),默认是INT 类型，也就是说一般情况下当你将整型数字存储Hive 表中的整型类型(`TINYINT`, `SMALLINT`, `INT/INTEGER`, `BIGINT`)的字段的时候，它所采用的的数据类型就是INT,而不是取决于你采用的到底是整型类型中的某一种，例如`TINYINT`和`BIGINT`

只有当你存入的数字超出了INT 类型的限制，它才可能采取你定义的`BIGINT`类型，或者你存储数据的时候强制指定了数据的类型，怎么强制指定呢，通过添加后缀的方式

| Type     | Postfix | Example |
| :------- | :------ | :------ |
| TINYINT  | Y       | 100Y    |
| SMALLINT | S       | 100S    |
| BIGINT   | L       | 100L    |

#### 浮点型

如果用来表示商品的单价等金额信息可以使用double数据类型，double可以保存双精度数值，float 是单精度类型。

但是这两种类型我们在Hive 中用的却不是很多，因为它会失去精度



#### DECIMAL和NUMERIC

一般企业中我们会指定数值的位数和精度，这个时候我们就需要使用DECIMAL数据类型，该类型可以指定任意的精度和位数，方便用于计算。用法是 `DECIMAL(precision, scale)` ，其中precision是精度，指定有多少位有效数字，scale是小数位数，指小数点后有多少位数字，比如DECIMAL(10,2)指的就是有10位有效数字，小数点后是2位。

Hive-0.11.0和Hive-0.12.0固定了DECIMAL类型的精度并限制为38位数字，从Hive-0.13.0开始可以指定DECIMAL的规模和精度，当使用DECIMAL类型创建表时可以使用DECIMAL(precision,scale)语法。DECIMAL(9,8)代表最多9位数字，后8位是小数。此时也就是说，小数点前最多有1位数字，如果超过一位则会变成null。如果不指定参数，那么默认是DECIMAL(10,0)，即没有小数位，此时0.82会变成1。

 DECIMAL类型比DOUBLE类型为浮点数提供了精确的数值和更广的范围，DECIMAL类型存储了数值的精确地表示，而DOUBLE类型存储了非常接近数值的近似值。当DOUBLE类型的近似值精度不够时可以使用DECIMAL类型，比如金融应用，等于和不等于检查以及舍入操作，当数值超出了DOUBLE类型的范围（< -10<sup>308</sup> or > 10<sup>308</sup>）或者非常接近于0（-10<sup>308</sup>> < ... < 10^<sup>308</sup>）时，也可以使用DECIMAL类型。

Hive 的 `DECIMAL` 类型依赖Java 的`BIGDECIMAL`，在Java 中它的意思是不可变的任意精度十进制数。多有对其他数字类型的操作对`DECIMAL`也同样适用,`DECIMAL`的持久化支持科学和非科学的计数法，也就是说如果你的数据文件里存储的是例如4.004E<sup>3</sup> 和 4004 这样格式的，对`BIGDECIMAL`来说都是合法的。

### 时间类型

| 类型     | 格式                          | 范围                  | 名称           | 引入版本          | 说明                                 |
| :------- | :---------------------------- | --------------------- | :------------- | ----------------- | ------------------------------------ |
| DATE     | yyyy-MM-dd                    | 0000.01.01~9999.12.31 | 字符串         | Hive 0.12.0中引入 | 主要用来表示年月日                   |
| IMESTAMP | yyyy-MM-dd HH:mm:ss.fffffffff | 是                    | 长度不定字符串 | Hive 0.8.0中引入  | 小数后为九位，也就是说精度为纳秒级别 |
| CHAR     | 最大的字符数：255             | 否                    | 长度固定字符串 | Hive 0.13.0中引入 | 'abc' or "abc"                       |

HIVE中提供DATE类型描述年月日，格式为yyyy-MM-dd。 范围为0000.01.01~9999.12.31。

HIVE中提供TIMESTAMP类型描述细致时间类型，是带有时区信息的

hive中的日期类型不常用，我们一般将日期类型的数据保存为string类型，也省去了日期格式转换的麻烦。字符串类型一般使用string类型，不推荐使用VARCHAR和CHAR类型。

上面我们也说了不建议使用时间类型统一使用字符串类型。但是如果已经有了的表存在了时间类型，你可以参考下面的转化方式进行转换



| Valid casts to/from Date type | Result                                                       |
| :---------------------------- | :----------------------------------------------------------- |
| cast(date as date)            | Same date value                                              |
| cast(timestamp as date)       | timestamp 转出 date                                          |
| cast(string as date)          | 如果字符串是的格式是 'YYYY-MM-DD'，你也可以将其转成date 类型，如果字符串不是'YYYY-MM-DD' 格式的则返回 NULL |
| cast(date as timestamp)       | date 类型转换成timestamp，时区信息是当前时区                 |
| cast(date as string)          | date类型转换成'YYYY-MM-DD'格式的字符串                       |



### 字符类型

| 类型    | 长度范围            | 末尾空格是否影响比较 | 名称           | 引入版本          | 例子           |
| :------ | :------------------ | -------------------- | :------------- | ----------------- | -------------- |
| STRING  | 不限长度            | 是                   | 字符串         | 一开始就有        | 'abc' or "abc" |
| VARCHAR | 字符数范围1 - 65535 | 是                   | 长度不定字符串 | Hive 0.12.0中引入 | 'abc' or "abc" |
| CHAR    | 最大的字符数：255   | 否                   | 长度固定字符串 | Hive 0.13.0中引入 | 'abc' or "abc" |

`STRING`类型在Hive 中可以使用单引号或者双引号括起来表示，”“ 或者 ''

`VARCHAR` 类型可以认为是`STRING`类型的长度限定版本，它只能存储的字符长度在1 到 65535 ，如果你将`STRING`转化为`VARCHAR`类型的时候，如果长度超出了限制就会被截断，和`STRING`类型一样两端的空格非常重要，会影响字符串的比较结果

非泛型UDF不能直接使用varchar类型作为输入参数或返回值。可以创建字符串UDF，而varchar值将转换为字符串并传递给UDF。要直接使用varchar参数或返回varchar值，请创建一个GenericUDF。如果其他上下文依赖于基于反射的方法来检索类型信息，那么可能还有其他不支持varchar的上下文。这包括一些SerDe实现。

`CHAR`类型和`VARCHAR`类型相似，不同的实`VARCHAR`是长度可变的，变化范围是1 到 65535 ，但是`CHAR`类型长度是不可变的，也就是你在创建的时候需要指定长度，如果数据长度不及指定长度则会在末尾添加空格补齐，但是这个长度不影响比较，可支持的最大长度是255

很多人以为时间戳应该是一个整型数字，其实不是的，在数据库里面的时间戳其实是可读的，也就是`yyyy-MM-dd HH:mm:ss.fffffffff` 这个形式的



#### Intervals

sql计算时，一般我们来做时间加减会unix_timestamp和from_unixtime结合使用：先把时间unix距1970年至今的整数秒，再进行加减就达到了时间前后的目的,现在我们来认识一个在时间加减上更为简便的函数：**INTERVAL**

```sql
-- 当前时间十秒前的时间
select current_timestamp() - INTERVAL 10 second;
-- 1年后的时间 
select current_timestamp()  + INTERVAL 1 year;
```

还可以按照年月，月日，日时，时分，分秒的格式来加减，格式是以任意分隔符(一般是空格)，对应的是year_month,month_day等，需要注意的是必须是相邻的，且大单位在前，比如只能是年月(year_month)，不能是月年，更不能是年时，年分等

```sql
-- 比如一年一个月前
select current_timestamp()  - INTERVAL '1 1' year_month
```

这里你可以将 `current_timestamp()` 换成任意时间函数，例如

```
select current_date()  + INTERVAL 1 year;
```



### 其他类型

`BOOLEAN` 布尔类型 表示真假 

`BINARY`  字节数组 我就没用过

对于boolean类型，我们一般用0和1来代替，后续的使用处理上也会方便一些。



## 类型转换

Hive 的原子数据类型是可以进行隐式转换的，类似于 Java 的类型转换，例如某表达式使用 INT 类型，TINYINT 会自动转换为 INT 类型，但是 Hive 不会进行反向转化，例如，某表达式使用 TINYINT 类型，INT 不会自动转换为 TINYINT 类型，它会返回错误，除非使用 CAST 操作。



**隐式类型转换规则如下**

任何整数类型都可以隐式地转换为一个范围更广的类型，如 TINYINT 可以转换成 INT，INT 可以转换成 BIGINT。

所有整数类型、FLOAT 和 STRING 类型都可以隐式地转换成 DOUBLE。

TINYINT、SMALLINT、INT 都可以转换为 FLOAT。

BOOLEAN 类型不可以转换为任何其它的类型。



**可以使用 CAST 操作显示进行数据类型转换**

例如 CAST('1' AS INT)将把字符串'1' 转换成整数 1；如果强制类型转换失败，如执行CAST('X' AS INT)，表达式返回空值 NULL。



下面我们提供了一个类型转换表，表里的true 就代表行的可以转化成对应列上的类型

|              | void  | boolean | tinyint | smallint | int   | bigint | float | double | decimal | string | varchar | timestamp | date  | binary |
| :----------- | :---- | :------ | :------ | :------- | :---- | :----- | :---- | :----- | :------ | :----- | :------ | :-------- | :---- | :----- |
| void to      | true  | true    | true    | true     | true  | true   | true  | true   | true    | true   | true    | true      | true  | true   |
| boolean to   | false | true    | false   | false    | false | false  | false | false  | false   | false  | false   | false     | false | false  |
| tinyint to   | false | false   | true    | true     | true  | true   | true  | true   | true    | true   | true    | false     | false | false  |
| smallint to  | false | false   | false   | true     | true  | true   | true  | true   | true    | true   | true    | false     | false | false  |
| int to       | false | false   | false   | false    | true  | true   | true  | true   | true    | true   | true    | false     | false | false  |
| bigint to    | false | false   | false   | false    | false | true   | true  | true   | true    | true   | true    | false     | false | false  |
| float to     | false | false   | false   | false    | false | false  | true  | true   | true    | true   | true    | false     | false | false  |
| double to    | false | false   | false   | false    | false | false  | false | true   | true    | true   | true    | false     | false | false  |
| decimal to   | false | false   | false   | false    | false | false  | false | false  | true    | true   | true    | false     | false | false  |
| string to    | false | false   | false   | false    | false | false  | false | true   | true    | true   | true    | false     | false | false  |
| varchar to   | false | false   | false   | false    | false | false  | false | true   | true    | true   | true    | false     | false | false  |
| timestamp to | false | false   | false   | false    | false | false  | false | false  | false   | true   | true    | true      | false | false  |
| date to      | false | false   | false   | false    | false | false  | false | false  | false   | true   | true    | false     | true  | false  |
| binary to    | false | false   | false   | false    | false | false  | false | false  | false   | false  | false   | false     | false | true   |



