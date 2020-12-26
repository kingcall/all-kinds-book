## 1.like的使用详解

**1.语法规则**:

1. **格式是A like B,其中A是字符串，B是表达式，****表示能否用B去完全匹配A的内容，换句话说能否用B这个表达式去表示A的全部内容，注意这个和rlike是有区别的****。返回的结果是True/False.**
2. **B只能使用简单匹配符号 _和%，”_”表示任意单个字符，字符”%”表示任意数量的字符**
3. **like的匹配是按字符逐一匹配的，使用B从A的第一个字符开始匹配，所以即使有一个字符不同都不行。**

**2.操作类型: strings**
**3.使用描述**: ***\*如果字符串A或者字符串B为NULL，则返回NULL；如果字符串A符合表达式B  的正则语法，则为TRUE；否则为FALSE。尤其注意NULL值的匹配，返回的结果不是FALSE和TRUE,而是null，其实除了is null ,is not null，其他的关系运算符只要碰到null值出现，结果都是返回NULL,而不是TRUE/FALSE。\****

```sql
hive (default)> select  'abcde'  like 'abc';
OK
false
hive (default)> select  null   like '%';
OK
NULL
hive (default)> select  'abc'   like null ;
OK
NULL
```

**4.案例演示**：

**'foobar' LIKE 'foo'的值为FALSE，而'foobar' LIKE 'foo___'的值为TRUE， 'foobar' LIKE 'foo%'的值为TRUE。要转义%，请使用\(%匹配一个%字符)。如果数据包含分号，你想匹配它，则需要转义，像'a\;b'**

```sql
hive (default)> select  'abcde'  like 'abc';
OK
false
hive (default)> select  'abcde'  like 'abc__';
OK
true
hive (default)> select  'abcde'  like 'abc%';
OK
true
hive (default)> select  'abcde'  like '%abc%';
OK
true
hive (default)> select  'abcde'  like 'bc%';
OK
false
hive (default)> select  'abcde'  like '_bc%';
OK
true
hive (default)> select  'abcde'  like '_b%';
OK
true
```

**5.注意事项****：否定比较时候用NOT A LIKE B（使用A NOT LIIKE B也可以）,结果与like的结果时相对的。****当然前提要排除出现null问题，null值这个奇葩除外。**

```sql
hive (default)> select  'abcde'  like 'abc';
OK
false
hive (default)> select  not   'abcde'  like 'abc';
OK
true
hive (default)> select    'abcde' not   like 'abc';
OK
true
hive (default)> select null like '%';
OK
NULL
hive (default)> select not  null like '%';
OK
NULL
hive (default)> select   null not  like '%';
OK
NULL
```

## 2. RLIKE比较符使用详解

**1.语法规则:**

1. ***\*A RLIKE B ，表示B是否在A里面即可。而A LIKE B,则表示B是否是A.\****
2. ***\*B中的表达式可以使用JAVA中全部正则表达式，具体正则规则参考java，或者其他标准正则语法。\****

**2.操作类型**: strings
**3.使用描述: \**如果字符串A或者字符串B为NULL，则返回NULL；如果字符串A符合JAVA正则表达式B的正则语法，则为TRUE；否则为FALSE。\****

```sql
hive (default)> select 'footbar' rlike '^f..]+r$';
OK
false
hive (default)> select 'footbar' rlike '^f.*r$';
OK
true
hive (default)> select 'foobar' rlike 'foo';  --注意同样表达式，用正则匹配成功
OK
true
hive (default)> select 'foobar' like 'foo';--注意同样表达式，用like匹配失败
OK
false
hive (default)> select '123456' rlike '^\\d+$';
OK
true
hive (default)> select null rlike '.*';
OK
NULL
```

### 3.NOT A LIKE B 与 A not like B

 **1. not..like是like的否定用法，如果like匹配结果时true，则not..like的匹配结果时false，反之也是结果也是相对。****当然前提要排除出现null问题，null值这个奇葩除外，null的结果都是null值。**

```sql
hive> select 1 from t_fin_demo  where NOT 'football' like 'fff%';
hive>select 1 from t_fin_demo where 'football' not  like 'fff%';
hive> select 1 from t_fin_demo where 'football'  like 'fff%';
```

### 4.关于like与rlike,not like,like not的使用对比总结

  **1.Rlike*****\*功能和like功能大致一样，like是后面只支持简单表达式匹配（_%）,而rlike则支持标准正则表达式语法。所以如果正则表达式使用熟练的话，建议使用rlike，功能更加强大。所有的like匹配都可以被替换成rlike。反之，则不行。\******但是注意：like是从头逐一字符匹配的，是全部匹配，但是rlike则不是,可以从任意部位匹配，而且不是全部匹配。**

```sql
hive (default)> select 'foobar' like 'foo';
OK
false
hive (default)> select 'foobar' like 'foo';
OK
false
hive (default)> select 'foobar' like 'oo%';
OK
false
hive (default)> select 'foobar' rlike 'foo';
OK
true
hive (default)> select 'foobar' rlike '.oo.*';
OK
true
```

  ***\*2\**\**.\**** **NOT A LIKE B是LIKE的结果否定，如果like匹配结果时true，则not..like的匹配结果时false，反之也是结果也是相对。实际中也可以使用 A NOT LIKE B，也是LIKE的否定，与 NOT A LIKE B一样。当然前提要排除出现null问题，null值这个奇葩除外，null的结果都是null值。**

  ***\*3.同理NOT RLIKE 的使用，也是NOT A RLIKE B是对RLIKE的否定。\******当然前提要排除出现null问题，null值这个奇葩除外，null的结果都是null值。**

### **5.regexp的用法和rlike一样**

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/26/11:16:23-watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2NDQyNTUz,size_16,color_FFFFFF,t_70.png)