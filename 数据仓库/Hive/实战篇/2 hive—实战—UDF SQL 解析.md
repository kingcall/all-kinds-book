[TOC]

## 场景

`UDF`在`hive`中使用场景广泛，这里列举常用的使用场景。

### IP 转化为地址

落到ods层的用户日志里面通常都有一个ip字段，在进入dws层的时候，通常数据清洗需要将其转化为类似中国-湖北-武汉的格式，便于后面进行地域维度的统计分析，或者在查询的时候直接使用udf函数转化。

需要去这种服务商购买Ip-地址映射库，然后用他们给出的`demo`进行解析，这里分享一份：

```java
@Test
public void test2() throws IOException, IPv4FormatException {
    City city = new City("src/file/mydata4vipday2.datx");
    System.out.println(city.find("180.169.26.98"));
    String[] locations = city.find("122.115.230.26");
    for(String location : locations){
        System.out.println(location);
    }
}
```

在编写这种ip转地区的udf的时候，就可以按照demo里面的调用方式来写，然后将其注册为永久函数，便于后面使用。

当然如果你没有购买服务商的服务，那你可以在网上找免费的资源，然后进行使用，这个时候可能存在的问题就是解析是否准确，使用是否方便，这里我提供一个自己觉得还行的一个资源





