[toc]
	com.facebook.presto.server.PrestoServer	Unable to create injector, see the following errors:

1) Configuration property 'query.max-memory=5GB' was not used
  at io.airlift.bootstrap.Bootstrap$5.configure(Bootstrap.java:278)

## 配置属性
### node.properties
```
node.environment=production
node.id=ffffffff-ffff-ffff-ffff-ffffffffffff
node.data-dir=/var/lib/kylin/prestodata
```
### jvm.config
```
-server
-Xmx16G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:OnOutOfMemoryError=kill -9 %p
```

### config.properties
#### coordinator
```
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8080
query.max-memory=50GB
query.max-memory-per-node=1GB
discovery-server.enabled=true
discovery.uri=http://example.net:8080
```
#### worker
```
coordinator=false
http-server.http.port=8080
query.max-memory=50GB
query.max-memory-per-node=1GB
discovery.uri=http://example.net:8080
```
#### 单台机器测试
```
coordinator=true
node-scheduler.include-coordinator=true
# 是否允许在coordinator服务中进行调度工作。对于大型的集群，在一个节点上的Presto server即作为coordinator又作为worker将会降低查询性能。因为如果一个服务器作为worker使用，大部分的资源都会被worker占用，那么就不会有足够的资源进行关键任务调度、管理和监控查询执行。
http-server.http.port=8080
# Presto 使用 HTTP进行内部和外部的所有通讯。
query.max-memory=5GB
query.max-memory-per-node=1GB
# 一个单独的查询任务使用任何一个节点的最大内存。
discovery-server.enabled=true
# Presto 通过Discovery 服务来找到集群中所有的节点。为了能够找到集群中所有的节点，每一个Presto实例都会在启动的时候将自己注册到discovery服务。
# Presto为了简化部署，并且也不想再增加一个新的服务进程，Presto coordinator 可以运行一个内嵌在coordinator 里面的Discovery 服务。这个内嵌的Discovery 服务和Presto共享HTTP server并且使用同样的端口。
discovery.uri=http://example.net:8080
```

##  运行Presto
- bin/launcher start  后台服务的方式启动
- bin/launcher run    前台服务的方式启动可以查看日志
- bin/laucher stop

![image-20210222114224631](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210222114224631.png)

## 客户端
- 下载客户端jar包
- 重命名成presto 放在presto 安装的目录下的bin 文件夹
- chmod +x presto 赋予执行全新安
- presto --server localhost:18081 --catalog hive --schema ods

localhost:18081 cordinator 的地址

![image-20210222114302616](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210222114302616.png)

### 官方客户端
### 可视化客户端

### 整合其他
#### superset

#### zeeplin



