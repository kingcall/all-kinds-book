[TOC]
## 1 概述

Redis 3.2 中增加了对GEO类型的支持。GEO，Geographic，地理信息的缩写。该类型，就是元素的2维坐标，在地图上就是经纬度。redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度Hash等常见操作。

通过 `help @geo` 可以看到全部的操作支持：

```shell
127.0.0.1:6379> help @geo
GEOADD key longitude latitude member [longitude latitude member ...]
添加成员的经纬度信息

GEODIST key member1 member2 [unit]
计算成员间距离

GEOHASH key member [member ...]
计算经纬度Hash

GEOPOS key member [member ...]
获取经纬度

GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
基于经纬度坐标范围查询

GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
基于成员范围查询
```


下面对常用操作，做说明演示。

## 2 GEOADD，添加成员的经纬度信息

语法：

> GEOADD key 经度 纬度 member

以吉林省主要城市的经纬度为例：

```shell
>  geoadd citys 125.19 43.54 changchun
>  geoadd citys 122.50 45.38 baicheng
>  geoadd citys 126.26 41.56 baishan
>  geoadd citys 124.18 45.30 daan
>  geoadd citys 125.42 44.32 dehui 
>  geoadd citys 128.13 43.22 dunhua
>  geoadd citys 124.49 43.31 gongzhuling 
>  geoadd citys 129.00 42.32 helong
>  geoadd citys 126.44 42.58 huadian
>  geoadd citys 130.22 42.52 hunchun
>  geoadd citys 126.11 41.08 jian
>  geoadd citys 127.21 43.42 jiaohe 
>  geoadd citys 126.33 43.52 jilin
>  geoadd citys 125.51 44.09 jiutai
>  geoadd citys 125.09 42.54 liaoyuan
>  geoadd citys 126.53 41.49 linjiang
>  geoadd citys 129.26 42.46 longjing
>  geoadd citys 125.40 42.32 meihekou
>  geoadd citys 126.57 44.24 shulan
>  geoadd citys 124.22 43.10 siping
>  geoadd citys 124.49 45.11 songyuan
>  geoadd citys 122.47 45.20 taoyan
>  geoadd citys 125.56 41.43 tonghua
>  geoadd citys 129.51 42.57 tumen
>  geoadd citys 129.30 42.54 yanjin
>  geoadd citys 126.32 44.49 yushu
```


## 3 GEODIST，计算成员间距离

语法：

> GEODIST key member1 member2 [unit]

unit 为结果单位，可选，支持：m，km，mi，ft，分别表示米（默认），千米，英里，英尺。

计算演示，计算长春到敦化的距离：

```shell
>  GEODIST citys changchun dunhua
"240309.2820"

>  GEODIST citys changchun dunhua km
"240.3093"
```

## 4 GEORADIUS 基于经纬度坐标的范围查询

语法：

> GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]

检索以某个经纬度为圆心，在特定半径的圆形范围内的成员。

查询演示，以经纬度125，42为圆心，100公里范围内的成员：

```shell
>  GEORADIUS citys 125 42 100 km
1) "tonghua"
2) "meihekou"
3) "liaoyuan"
```

支持的可选项的意义是：

- WITHCOORD，同时获取成员经纬度
- WITHDIST，同时获取距离参考点（圆心）的距离
- WITHHASH，同时获取成员经纬度HASH值
- COUNT count，限制获取成员的数量
- ASC|DESC，结果升降序排序
- STORE key，在命令表，READONLY模式下使用
- STOREDIST key，在命令表，READONLY模式下使用

## 5 GEORADIUSBYMEMBER 基于成员位置范围查询

语法：

> GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST ke##y]

检索以某个成员为圆心，在特定半径的圆形范围内的成员。功能与 GEORADIUS 类似，只不过圆心为某个成员位置。

查询演示，以经纬度 changchun 为圆心，100公里范围内的成员：

```shell
> GEORADIUSBYMEMBER citys changchun 100 km
1) "siping"
2) "gongzhuling"
3) "changchun"
4) "jilin"
5) "jiutai"
6) "dehui"
```


## 6 GEOPOS，获取成员经纬度

语法：

> GEOPOS key member [member ...]

获取某个成员经纬度：

```shell
> GEOPOS citys changchun
1) "125.19000023603439"
2) "43.539999086145414"
```

## 7 GEOHASH 计算经纬度Hash

语法：

> GEOHASH key member [member ...]

获取将经纬度坐标生成的HASH字符串。

```shell
> GEOHASH citys changchun
1) "wz9p8y0wfk0"
```


GEOHASH，是表示坐标的一种方法，便于检索，存储。

以上就是 Redis 提供的对2D位置信息的支持。