

[TOC]



Codis 是豌豆荚公司开发的一个分布式 Redis 解决方案，用Go语言开发的。对于上层的应用来说，连接到 Codis Proxy 和连接原生的 Redis Server 没有明显的区别 （不支持的命令列表），Codis 底层会处理请求的转发，不停机的数据迁移等工作。所有后边的一切事情，对于前面的客户端来说是透明的，可以简单的认为后边连接的是一个内存无限大的 Redis 服务。

## Codis 由四部分组成:

- Codis Proxy (codis-proxy)，处理客户端请求，支持Redis协议，因此客户端访问Codis Proxy跟访问原生Redis没有什么区别；
- Codis Dashboard (codis-config)，Codis 的管理工具，支持添加/删除 Redis 节点、添加/删除 Proxy 节点，发起数据迁移等操作。codis-config 本身还自带了一个 http server，会启动一个 dashboard，用户可以直接在浏览器上观察 Codis 集群的运行状态；
- Codis Redis (codis-server)，Codis 项目维护的一个 Redis 分支，基于 2.8.21 开发，加入了 slot 的支持和原子的数据迁移指令；
- ZooKeeper/Etcd，Codis 依赖 ZooKeeper 来存放数据路由表和 codis-proxy 节点的元信息，codis-config 发起的命令都会通过 ZooKeeper 同步到各个存活的 codis-proxy；

## codis和twemproxy最大的区别有两个：

- codis支持动态水平扩展，对client完全透明不影响服务的情况下可以完成增减redis实例的操作；
- codis是用go语言写的并支持多线程，twemproxy用C并只用单线程。 后者又意味着：codis在多核机器上的性能会好于twemproxy；codis的最坏响应时间可能会因为GC的STW而变大，不过go1.5发布后会显著降低STW的时间；如果只用一个CPU的话go语言的性能不如C，因此在一些短连接而非长连接的场景中，整个系统的瓶颈可能变成accept新tcp连接的速度，这时codis的性能可能会差于twemproxy。


## codis和redis cluster的区别：

- redis cluster基于smart client和无中心的设计，client必须按key的哈希将请求直接发送到对应的节点。这意味着：使用官方cluster必须要等对应语言的redis driver对cluster支持的开发和不断成熟；client不能直接像单机一样使用pipeline来提高效率，想同时执行多个请求来提速必须在client端自行实现异步逻辑。 而codis因其有中心节点、基于proxy的设计，对client来说可以像对单机redis一样去操作proxy（除了一些命令不支持），还可以继续使用pipeline并且如果后台redis有多个的话速度会显著快于单redis的pipeline。同时codis使用zookeeper来作为辅助，这意味着单纯对于redis集群来说需要额外的机器搭zk，不过对于很多已经在其他服务上用了zk的公司来说这不是问题：）

## coids HA
因为codis的proxy是无状态的，可以比较容易的搭多个proxy来实现高可用性并横向扩容。

对Java用户来说，可以使用经过我们修改过的Jedis，Jodis ，来实现proxy层的HA。它会通过监控zk上的注册信息来实时获得当前可用的proxy列表，既可以保证高可用性，也可以通过轮流请求所有的proxy实现负载均衡。如果需要异步请求，可以使用我们基于Netty开发的Nedis。

对下层的redis实例来说，当一个group的master挂掉的时候，应该让管理员清楚，并手动的操作，因为这涉及到了数据一致性等问题（redis的主从同步是最终一致性的）。因此codis不会自动的将某个slave升级成master。 不过我们也提供一种解决方案：codis-ha。这是一个通过codis开放的api实现自动切换主从的工具。该工具会在检测到master挂掉的时候将其下线并选择其中一个slave提升为master继续提供服务。

==需要注意==，codis将其中一个slave升级为master时，该组内其他slave实例是不会自动改变状态的，这些slave仍将试图从旧的master上同步数据，因而会导致组内新的master和其他slave之间的数据不一致。因为redis的slave of命令切换master时会丢弃slave上的全部数据，从新master完整同步，会消耗master资源。因此建议在知情的情况下手动操作。使用 codis-config server add <group_id> <redis_addr> slave 命令刷新这些节点的状态即可。codis-ha不会自动刷新其他slave的状态。

## 与Twemproxy Redis Cluster对比
![image](FB4A490553D7436299C35D886B511DD1)

## Codis 分片原理

- Codis 将所有的 key 默认划分为 1024 个槽位(slot),每个槽位唯一映射唯一Redis；
- Codis 在内存维护槽位和 Redis 实例的映射关系
    - 1.对客户端传过来的 key 进行 crc32 运算计算哈希值
    - 2.Hash 后的整数值对 1024 这个整数进行取模得到一个余数，这个余数就是对应 key 的槽位
			hash = crc32(command.key) 
			slot_index = hash % 1024 
			redis = slots[slot_index].redis 
			redis.do(command)



## Jodis 与 HA
因为 codis-proxy 是无状态的，可以比较容易的搭多个实例，达到高可用性和横向扩展。

对 Java 用户来说，可以使用基于 Jedis 的实现 Jodis ，来实现 proxy 层的 HA：

- 它会通过监控 zookeeper 上的注册信息来实时获得当前可用的 proxy 列表，既可以保证高可用性；
- 也可以通过轮流请求所有的proxy实现负载均衡。

如果需要异步请求，可以使用我们基于Netty开发的 Nedis。

需要的依赖
```
<!-- https://mvnrepository.com/artifact/com.wandoulabs.jodis/jodis -->
<dependency>
    <groupId>com.wandoulabs.jodis</groupId>
    <artifactId>jodis</artifactId>
    <version>0.2.2</version>
</dependency>
```

java 代码：
```
public static void main(String[] args) {

    // codis 连接池配置
    JedisPoolConfig jps = new JedisPoolConfig();
    jps.setMaxIdle(5);
    jps.setMinIdle(10);
    jps.setMaxTotal(16);
    
    JedisResourcePool jedisPool = RoundRobinJedisPool
            .create()
            .poolConfig(jps)
            .curatorClient("212.64.50.17:2181,212.64.72.219:2181,212.64.70.99:2181", 30000)
            .zkProxyDir("/jodis/codis-bigdata")
            .password("TKJeyui20i454")
            .build();
    try (Jedis jedis = jedisPool.getResource()) {
        String values = jedis.get("{37305908-8007-4535-B243-F3AC1125C45E}_wf");
        System.out.println(values);
    }
}
```
对下层的 redis 实例来说，当一个 group 的 master 挂掉的时候，应该让管理员清楚，并手动的操作，因为这涉及到了数据一致性等问题（redis的主从同步是最终一致性的）。因此 codis 不会自动的将某个 slave 升级成 master。关于外部 codis-ha 工具（具体可以参考之前的章节），这是一个通过 codis-dashboard 开放的 RESTful API 实现自动切换主从的工具。该工具会在检测到 master 挂掉的时候主动应用主从切换策略，提升单个 slave 成为新的 master。

需要注意，codis 将其中一个 slave 升级为 master 时，该组内其他 slave 实例是不会自动改变状态的，这些 slave 仍将试图从旧的 master 上同步数据，因而会导致组内新的 master 和其他 slave 之间的数据不一致。因此当出现主从切换时，需要管理员手动创建新的 sync action 来完成新 master 与 slave 之间的数据同步（codis-ha 不提供自动操作的工具，因为这样太不安全