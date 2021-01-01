

[TOC]

## Hive 中的匹配

为什么我们说是Hive 中的匹配，没有说是Hive 中的正则匹配呢？因为这里的匹配除了正则的，还有不是正则的，SQL 提供的简化版本的匹配那就是like ，特点即使使用起来简单，当然功能也相对简单，所以相对正则来说还是简单那么一丢丢的，在一些简单的场景下就可以使用，也正是体现了杀鸡焉用牛刀的优良传统。

like不是正则匹配。关于like可以看一下SQL的标准，例如%代表任意多个字符 _ 代表单个字符。rlike是正则，正则的写法与java一样。'\'需要使用'\\',例如'\w'需要使用'\\w'

正则表达式是描述搜索模式的特殊字符串。 它是一个强大的工具，为我们提供一种简洁灵活的方法来识别基于模式的文本字符，例如字符，单词等。例如，我们可以使用正则表达式来搜索电子邮件，IP地址，电话号码，社会安全号码或具有特定模式的任何内容。正则表达式可以由正则表达式处理器解释的自己的语法，正则表达式广泛应用于从编程语言到数据库(包括MySQL)大部分平台。使用正则表达式的优点是，不限于在like运算符中基于具有百分号(`%`)和下划线(`_`)的固定模式搜索字符串。 使用正则表达式，有更多的元字符来构造灵活的模式。

### 1. like的使用详解

**1.语法规则**:

1. **格式是A like B,其中A是字符串，B是表达式**，表示能否用B去完全匹配A的内容，换句话说能否用B这个表达式去表示A的全部内容，注意这个和rlike是有区别的。返回的结果是True/False
2. **B只能使用简单匹配符号 _和%，\_表示任意单个字符，字符”%”表示任意数量的字符**
3. **like的匹配是按字符逐一匹配的，使用B从A的第一个字符开始匹配，所以即使有一个字符不同都不行。**

**2.操作类型: strings**
**3.使用描述**: 如果字符串A或者字符串B为NULL，则返回NULL；如果字符串A符合表达式B  的正则语法，则为TRUE；否则为FALSE。尤其注意NULL值的匹配，返回的结果不是FALSE和TRUE,而是null，其实除了is null ,is not null，其他的关系运算符只要碰到null值出现，结果都是返回NULL,而不是TRUE/FALSE

```sql
0: jdbc:hive2://localhost:10000> select  'abcde'  like 'abc';
OK
false
0: jdbc:hive2://localhost:10000> select  null   like '%';
OK
NULL
0: jdbc:hive2://localhost:10000> select  'abc'   like null ;
OK
NULL
```

**4.案例演示**：

**'foobar' LIKE 'foo'的值为FALSE，而'foobar' LIKE 'foo___'的值为TRUE， 'foobar' LIKE 'foo%'的值为TRUE。要转义%，请使用\(%匹配一个%字符)。如果数据包含分号，你想匹配它，则需要转义，像'a\;b'**

```sql
0: jdbc:hive2://localhost:10000> select  'abcde'  like 'abc';
false

0: jdbc:hive2://localhost:10000> select  'abcde'  like 'abc__';
true

0: jdbc:hive2://localhost:10000> select  'abcde'  like 'abc%';
true

0: jdbc:hive2://localhost:10000>select  'abcde'  like '%abc%';
true

0: jdbc:hive2://localhost:10000> select  'abcde'  like 'bc%';
false

0: jdbc:hive2://localhost:10000> select  'abcde'  like '_bc%';
true

0: jdbc:hive2://localhost:10000> select  'abcde'  like '_b%';
true
```

这里我们再演示一下如何去匹配% 和 _ ,因为我们是要去匹配它，所以这两个字符都是出现在被匹配字符串里的，而不是规则串里，所以我们只需要将其当做普通字符匹配

```sql
0: jdbc:hive2://localhost:10000> select  'ab%cde'  like 'ab_c%';
true

select  'ab%cde'  like 'ab%c%';
true
```

可以看到我们的% 和 _,在被匹配字符串里的时候就是一个普通字符，已经失去了在规则串中的含义，所以可以使用 % 和 _ 直接匹配，但是这里我们是将其当做字符进行匹配的，但是我们如何精准匹配呢，什么意思呢就是上面我们的规则串`ab%c%`

不止可以匹配'ab%cde'也可以匹配'abXcde' 啊，这里你把`X` 缓存任意的都成立,也就是让我们的规则串中的% 失去原来的意义，只匹配它本来的含义，也就是我们说的转移了，只需要前面添加一个\ 就可以了

```sql
-- 没有转义
0: jdbc:hive2://localhost:10000> select  'abXcde'  like 'ab%c%';
true
-- 转义
0: jdbc:hive2://localhost:10000> select  'abXcde'  like 'ab\%c%';
false
-- 这下就达到我们的目的了
0: jdbc:hive2://localhost:10000> select  'ab%cde'  like 'ab\%c%';
true
```

_ 的转义也是一样的，就不再在这里演示了



**5.注意事项****：否定比较时候用NOT A LIKE B（使用A NOT LIIKE B也可以）,结果与like的结果时相对的。null 参与的like 和 not like 返回值都是null

```sql
0: jdbc:hive2://localhost:10000> select 'abcde'  like 'abc';
false

0: jdbc:hive2://localhost:10000> select 'abcde' not like 'abc';
true

0: jdbc:hive2://localhost:10000> select 'abcde' not like 'abc';
true

0: jdbc:hive2://localhost:10000> select null like '%';
NULL

0: jdbc:hive2://localhost:10000> select not null like '%';
NULL

0: jdbc:hive2://localhost:10000> select null not  like '%';
NULL

0: jdbc:hive2://localhost:10000> select null like null;
NULL

0: jdbc:hive2://localhost:10000> select null not like null;
NULL
```

NULL 简直就是个bug 一般的存在，自己都不像自己，其实也可以理解因为其他值都是确定，而唯独NULL代表了一切不合理、不正确的结果(强行解释和加戏，估计导演不开心了)

### 2. RLIKE使用详解

**1.语法规则:**

1. **A RLIKE B** 表示的含义和 **A LIKE B**不太 一样，A LIKE B 表示A 是否和B 的规则匹配的上，**A RLIKE B** 表示的是A 是否含有B, 关键字从Like 换成rlike 了所以含义发生了一点变化,R 是正则regular 的缩写
2. 因为关键字是rlike 了，所以**B中的表达式可以使用JAVA中全部正则表达式，具体正则规则参考java，或者其他标准正则语法**

**2.操作类型**: strings
**3.使用描述**: 如果字符串A或者字符串B为NULL，则返回NULL；如果字符串A符合JAVA正则表达式B的正则语法，则为TRUE；否则为FALSE。

```sql
0: jdbc:hive2://localhost:10000> select 'footbar' rlike '^f..]+r$';
false

0: jdbc:hive2://localhost:10000> select 'footbar' rlike '^f.*r$';
true

--注意同样表达式，用正则匹配成功(是否包含)
0: jdbc:hive2://localhost:10000> select 'foobar' rlike 'foo'; 
true
--注意同样表达式，用like匹配失败(是否匹配得上)
hive (default)> select 'foobar' like 'foo';
false

0: jdbc:hive2://localhost:10000> select '123456' rlike '^\\d+$';
true

0: jdbc:hive2://localhost:10000>  select null rlike '.*';
NULL
```

其实我们可以看到rlike 的匹配功能更加强大，例如可以匹配数字，匹配字母，但是like 只能表达一种语义，那就是是否匹配得上，但是与之而来的就是你得熟悉正则的表达方式

### not A like B 与 A not like B

1. not like是like的否定用法，如果like匹配结果时true，则not like的匹配结果时false，反之也是结果也是相对。当然前提要排除出现null问题，null的结果都是null值。

```sql
hive> select 1 from t_fin_demo  where NOT 'football' like 'fff%';
hive>select 1 from t_fin_demo where 'football' not  like 'fff%';
hive> select 1 from t_fin_demo where 'football'  like 'fff%';
```

not A like B 与 A not like B  表达的含义是一样的，不用纠结，选择自喜欢的风格用就可以，我就只会用 not like ，而不是 not xxx like bbb

### 4. regexp 和rlike

regexp 和 rlike 是一样的，语法是一样的，含义也是一样的,不一样的就是名字了，regexp 看起来更像见名知意一些

![image-20201231104458044](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201231104458044.png)

## 总结

1. rlike 的匹配功能更加强大，例如可以匹配数字，匹配字母，但是like 不行，它只能表达一种语义，那就是是否匹配得上，

2. rlike 很强大但是与之而来的就是你得熟悉正则的表达方式，like 虽然功能简单，但是使用也简单，这就是为什么同时存在rlike 和 like 了

3. like是后面只支持简单表达式匹配（_%）,而rlike则支持标准正则表达式语法。所以如果正则表达式使用熟练的话，建议使用rlike，功能更加强大。所有的like匹配都可以被替换成rlike。反之，则不行。但是注意：like是从头逐一字符匹配的，是全部匹配，但是rlike则不是,可以从任意部位匹配，而且不是全部匹配

4.  NOT A LIKE B是LIKE的结果否定，如果like匹配结果时true，则not like的匹配结果时false，反之也是结果也是相对。实际中也可以使用 A NOT LIKE B，也是LIKE的否定，与 NOT A LIKE B一样。当然前提要排除出现null问题，null的结果不论怎样操作都是null。

5. 同理NOT RLIKE 的使用，也是NOT A RLIKE B是对RLIKE的否定。对null 值的处理和 like 以及 not like 一样

6. regexp 和rlike 是一样的，只不过名字不一样而已

   

