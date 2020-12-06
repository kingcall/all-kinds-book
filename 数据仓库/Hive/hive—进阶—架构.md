[toc]
## Metadata
- 元数据包含用Hive创建的database、table等的元信息。
- 元数据存储在关系型数据库中。如Derby、MySQL等。

### Metastore作用
- 客户端连接metastore服务，metastore再去连接MySQL数据库来存取元数据。有了metastore服务，就可以有多个客户端同时连接，而且这些客户端不需要知道MySQL数据库的用户名和密码，只需要连接metastore 服务即可。 

### 开启方式

#### 内嵌模式(不开启)
- 没有配置metaStore的时候,每当开启bin/hive;或者开启hiveServer2的时候,都会在内部启动一个metastore嵌入式服务,资源比较浪费,如果开启多个窗口,就会存在多个metastore server
> hive服务和metastore服务运行在同一个进程中，derby服务也运行在该进程中.内嵌模式使用的是内嵌的Derby数据库来存储元数据，也不需要额外起Metastore服务

> 这个是默认的，配置简单，但是一次只能一个客户端连接（这句话说实在有点坑，其实就是你启动一个hive服务会内嵌一个metastore服务，然后在启动一个又会内嵌一个metastore服务，并不是说你的客户端只能启动一个hive，是能启动多个，但是每个都有metastore，浪费资源），适用于用来实验，不适用于生产环境。

#### local mataStore(本地)
- 当metaStore和装载元数据的数据库(MySQL)存在同一机器上时配置是此模式,开启metastore服务就只需要开启一次就好,避免资源浪费!

#### Remote Metastore(远程)
- 当metaStore和装载元数据的数据库(MySQL)不存在同一机器上时配置是此模式,开启metastore服务就只需要开启一次就好,避免资源浪费