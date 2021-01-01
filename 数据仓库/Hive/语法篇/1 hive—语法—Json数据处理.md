

[TOC]

## Json 格式的数据处理

Json 数据格式是我们比较常用的的一种数据格式，例如埋点数据、业务端的数据、前后端调用都采用的是这种数据格式，所以我们很有必要学习一下这种数据格式的处理方法

### 准备数据

 cat json.data

```
{"movie":"1193","rate":"5","timeStamp":"978300760","uid":"1"}
{"movie":"661","rate":"3","timeStamp":"978302109","uid":"1"}
{"movie":"914","rate":"3","timeStamp":"978301968","uid":"1"}
{"movie":"3408","rate":"4","timeStamp":"978300275","uid":"1"}
{"movie":"2355","rate":"5","timeStamp":"978824291","uid":"1"}
{"movie":"1197","rate":"3","timeStamp":"978302268","uid":"1"}
{"movie":"1287","rate":"5","timeStamp":"978302039","uid":"1"}
{"movie":"2804","rate":"5","timeStamp":"978300719","uid":"1"}
{"movie":"594","rate":"4","timeStamp":"978302268","uid":"1"}
```

创建hive表并且加载数据

```sql
create table ods.ods_json_data(text string);
load data local inpath "/Users/XXX/workspace/hive/json.data" overwrite into table ods.ods_json_data;
```

### get_json_object 和 json_tuple 函数

json_tuple 不支持json 的嵌套处理，但是支持一次性获取多个顶级的key对应的值

get_json_object 不支持一次获取多个值，但是支持复杂json 的处理

#### get_json_object(）

用法：get_json_object(string json_string, string path)  前面我们介绍过如何查看函数的用法`desc function get_json_object`

返回值：String

说明：解析json的字符串json_string，返回path指定的内容。如果输入的json字符串无效，那么返回NUll，这个函数每次只能返回一个数据项。

具体示例： get_json_object(value,’$.id’)

`select get_json_object(text,"$.movie") from ods.ods_json_data;`

![image-20201230163733256](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230163733256.png)

这个函数的不足之处是，它只能返回一个值，就是我们不能一次性从json 中提取多个值，如果要提取多个值的话，就要多次调用这个函数,但是我们下面介绍的json_tuple 就可以,但是这不是说这个函数不强或者怎么样，记住这个函数的api 可以帮你节约很多时间

![image-20201230183648635](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230183648635.png)

#### json_tuple

用法：json_tuple(jsonStr, p1, p2, ..., pn) 整理的pn 就是我们要提取的键

返回值：tuple(v1,...vn) 这里的返回值v1 ... vn 和 键p1 .... pn 是相对应的

`select json_tuple(text,'movie','rate','timeStamp','uid') from ods.ods_json_data;`

![image-20201230163927109](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230163927109.png)

json_tuple相当于get_json_object的优势就是**一次可以解析多个Json字段**。

### 例子演示

#### 1. 嵌套json 的处理

前面我们说了json_tuple不支持嵌套JSON 的处理

```sql
select get_json_object('{"movie":"594","rate":"4","timeStamp":"978302268","uid":"1","info":{"name":"天之骄子"}}',"$.info.name")
select json_tuple('{"movie":"594","rate":"4","timeStamp":"978302268","uid":"1","info":{"name":"天之骄子"}}',"info.name")
```

![image-20201230175753453](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230175753453.png)

#### 2. Json数组解析(get_json_object 实现)

```
SELECT get_json_object('[{"website":"www.ikeguang.com","name":"我的生活记忆"},{"website":"beian.ikeguang.com","name":"备案"}]', '$.[0].website'), get_json_object('[{"website":"www.ikeguang.com","name":"我的生活记忆"},{"website":"beian.ikeguang.com","name":"备案"}]', '$.[1].website');
```

![image-20201230180054218](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230180054218.png)

这个时候时候你发现我提取的都是json 数组中的`website`,有没有什么简单的办法呢，理论上get_json_object 只能有一个返回值，无论如何都需要写多个，那你有没有想过一个问题，我要是这个数组里面有100个元素都是json,我需要每一个json 的website 那我是不是需要写100次了，这个时候你要是仔细阅读这个函数的api 的话，你就会发现了另外一个符号\*

```sql
SELECT get_json_object('[{"website":"www.ikeguang.com","name":"我的生活记忆"},{"website":"beian.ikeguang.com","name":"备案"}]', '$.[*].website')
```

![image-20201230183128589](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230183128589.png)

这下你知道了，get_json_object 是只能返回一个元素，不是只能返回一个字符串，上面本来就是一个json 数组，那要是我们是从json 里面解析出来的数组怎么处理呢？

```sql
SELECT get_json_object('{"info":[{"website":"www.ikeguang.com","name":"我的生活记忆"},{"website":"beian.ikeguang.com","name":"备案"}]}', '$.info');
```

![image-20201230181300897](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230181300897.png)

需要注意下面这样操作之后你拿到的就是一个json 字符串了，这下你就可以按照上面的方式再处理一次了

```sql
select get_json_object (get_json_object('{"info":[{"website":"www.ikeguang.com","name":"我的生活记忆"},{"website":"beian.ikeguang.com","name":"备案"}]}', '$.info' ),'$.[1].website');
```

![image-20201230182103977](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230182103977.png)

但是有时候我们希望直接获取，而不是通过这样嵌套的方式，这个时候其实就是将上面的嵌套的get_json_object函数的path 参数进行组合

```sql
SELECT get_json_object('{"info":[{"website":"www.ikeguang.com","name":"我的生活记忆"},{"website":"beian.ikeguang.com","name":"备案"}]}', '$.info[1].website');
```

![image-20201230182139617](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230182139617.png)

这个时候如果我们再上 \* 进行加持，那就很简单了

```sql
SELECT get_json_object('{"info":[{"website":"www.ikeguang.com","name":"我的生活记忆"},{"website":"beian.ikeguang.com","name":"备案"}]}', '$.info[*].website');
```

![image-20201230183443597](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230183443597.png)

其实到这里我们学习了指定一个数组的某个下标获取一个元素，指定\* 获取全部元素，那就如我就想获取前三个或者偶数个或者奇数个呢，哈哈，如果你回过头去看api 你就是知道了提供了一个`Union operator`，指定任意你想组合的下标即可,获取

```
SELECT get_json_object('{"info":[{"website":"www.ikeguang.com","name":"我的生活记忆"},{"website":"beian.ikeguang.com","name":"备案"}]}', '$.info[0,1].website');
```

![image-20201230184522813](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230184522813.png)

下面我们尝试获取一下偶数个，或者奇数个或者是一定范围内的奇数个或者偶数个,其实就是上面提供的数组切片，你可以参考api 进行使用

```
SELECT get_json_object('{"info":[{"website":"www.ikeguang.com","name":"我的生活记忆"},{"website":"beian.ikeguang.com","name":"备案"},{"website":"www.ikeguang2.com","name":"我的生活记忆"}]}', '$.info[0:2:2].website');
```

但是我尝试了一下，发现这个功能有bug，不能做到切片的效果，每次都是全部返回



### 加载JSON 数据

对于上面json.data 的数据，我们能不能在load 数据到hive 的时候就处理，而不是load 完之后再到使用的时候去处理，尤其是针对这种嵌套结构不是很复杂的这种json 格式

```sql
create table ods.ods_json_parse_data(
movie string,
rate string,
`timeStamp` string,
uid string)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS TEXTFILE;
load data local inpath "/Users/liuwenqiang/workspace/hive/json.data" overwrite into table ods.ods_json_parse_data;
```

![image-20201230195730821](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230195730821.png)

这种方法需要注意的是你的数据类型和字段名称都要匹配，否则就会报错或者不能获取到值,那要是复杂一点的嵌套结构呢，其实也可以,在上面的数据基础上添加了一个嵌套的字段也是可以的

```
{"movie":"1193","rate":"5","timeStamp":"978300760","uid":"1","info":{"name":"天之骄子"}}
```

```sql
create table ods.ods_json_parse_data2(
movie string,
rate string,
`timeStamp` string,
uid string,
info map<string,string>)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS TEXTFILE;
load data local inpath "/Users/liuwenqiang/workspace/hive/json.data" overwrite into table ods.ods_json_parse_data2;
```

![image-20201230201351438](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230201351438.png)

## 总结

1. get_json_object 和 json_tuple 函数的使用场景和其优缺点
2. 如果json 格式比较简单，那么可以在建表加载数据的时候就可以将json 处理掉，如果比较复杂也可以再加载的时候解析一部分，然后再通过SQL 进行解析
3. 也可以尝试写一些UDF 函数来处理JSON 

