## explode用法

在介绍如何处理之前，我们先来了解下`Hive`内置的 explode 函数，官方的解释是：`explode() takes in an array (or a map) as an input and outputs the elements of the array (map) as separate rows. UDTFs can be used in the SELECT expression list and as a part of LATERAL VIEW.` 意思就是 explode() 接收一个 array 或 map 类型的数据作为输入，然后将 array 或 map 里面的元素按照每行的形式输出。其可以配合 LATERAL VIEW 一起使用。光看文字描述很不直观，咱们来看看几个例子吧。

```sql
hive (default)> select explode(array('A','B','C'));
OK
A
B
C
Time taken: 4.188 seconds, Fetched: 3 row(s)
 
hive (default)> select explode(map('a', 1, 'b', 2, 'c', 3));
OK
key	value
a	1
b	2
c	3 
```

explode函数接收一个数组或者map类型的数据，通常需要用split函数生成数组。

这里有数据：

```
[{"AppName":"SogouExplorer_embedupdate","pepper":"-1"}]
[{"AppName":"SogouExplorer_embedupdate","pepper":"-1"}]
[{"AppName":"SogouExplorer_embedupdate","pepper":"-1"}]
[{"AppName":"2345Explorer_embedupdate","plugin":"-1"}]
[{"AppName":"SogouExplorer_embedupdate","pepper":"-1"}]
```

现在需要将AppName和pepper提取出来。

```sql
select json.appName, json.pepper, count(1) from (select explode(split(regexp_replace(regexp_replace(data, '\\}\\,\\{','\\}\\;\\{'),'\\[|\\]',''),'\\;')) as json from table)m2 lateral view json_tuple(json, 'AppName','pepper') json as appName, pepper group by json.appName, json.pepper;
```

这里解析json数组，我们本质上还是使用regexp_replace替换掉花括号和中括号，然后再使用split函数拆分为数据，给explode去分裂成多行。



## 解析json

通常需要在hive中使用sql解析json格式数据：

```sql
SELECT json_tuple('{"website":"www.ikeguang.com","name":"我的生活记忆"}', 'website', 'name');

www.ikeguang.com	我的生活记忆
```

json_tuple第一个参数要解析的json字段；后面跟要取的key的名称，有多个需要写多个，它不能解析json数组。

要解析json数组，也可以使用get_json_object函数。

```sql
SELECT get_json_object('[{"website":"www.ikeguang.com","name":"我的生活记忆"},{"website":"beian.ikeguang.com","name":"备案"}]', '$.[0].website');

www.ikeguang.com
```

如果我们想将整个 Json 数组里面的 website 字段都解析出来，如果这么写将非常麻烦，因为我们无法确定数组的长度，这个时候，上面的方法`使用regexp_replace替换掉花括号和中括号，然后再使用split函数拆分为数据`就是可以解决的。