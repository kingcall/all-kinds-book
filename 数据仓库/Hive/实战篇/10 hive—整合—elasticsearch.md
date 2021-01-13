配置过程如下：

1）添加jar包

```javascript
    add jar file:///path/to/elasticsearch-hadoop-hive-5.2.0.jar;
```

2）创建一个hive-es对应表

```javascript
create external table tmp.es_guo_test(
    imp_date string,
    group_code string,
    member_uin string,
    uin_flag bigint,
    ex_flag bigint,
    ower_flag bigint,
    xx_flag bigint
)STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler'
TBLPROPERTIES('es.nodes'='xxx','es.port'='9200','es.index.auto.create'='true','es.resource'='customer/guo_test','es.nodes.wan.only'='true');
```

1. 向外部表中导数据，则数据会自动写入ES中

```javascript
  insert overwrite table tmp.es_guo_test select imp_date,group_code,member_uin from tmp.guo_test where dt=20170813 limit 3000;
```

现在，查看ES索引，自动创建了customer索引，且有3000条数据。