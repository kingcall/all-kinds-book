[TOC]

## Ranger

Apache Ranger是一个框架，Hadoop上对于保护数据数据安全性的安全框架。用于在整个Hadoop平台上启用，监视和管理全面的数据安全性。支持多种大数据组件进行权限认证，以插件的形式对不同组件进行支持



Apache Ranger具有以下特性：

- 集中式安全管理，可在中央UI或使用REST API管理所有与安全相关的任务。
- 使用Hadoop组件/工具执行特定操作和/或操作的精细授权，并通过中央管理工具进行管理
- 跨所有Hadoop组件标准化授权方法。
- 增强了对不同授权方法的支持-基于角色的访问控制，基于属性的访问控制等。
- 在Hadoop的所有组件中集中审核用户访问和管理操作（与安全性相关）。



### 包含的组件如下：

- Ranger Admin 用户管理策略，提供WebUI和RestFul接口
- Ranger UserSync 用于将Unix系统或LDAP用户/组同步到RangerAdmin
- Ranger TagSync 同步Atlas中的Tag信息
- Ranger KMS



![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1205182-20201120153851778-193598173.png)



| 组件名称 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| Admin    | Ranger Admin Portal是安全管理的中心接口。 用户可以创建和更新策略，这些策略存储在策略数据库中。 每个组件内的Plugins会定期轮询这些策略。Portal还包括一个审计服务器，它发送从插件收集的审计数据，以便存储在HDFS或关系数据库中 |
| UserSync | 同步实用工具来从Unix或LDAP或Active Directory中拉取用户和组。 用户或组信息存储在Ranger门户中，用于策略定义。 |
| PlugIn   | 插件是嵌入每个集群组件进程的轻量级Java程序。 例如，Apache Hive的Apache Ranger插件嵌入在Hiveserver2中。 这些插件从中央服务器提取策略，并将它们本地存储在一个文件中。 当用户请求通过组件时，这些插件拦截请求并根据安全策略进行评估。 插件还可以从用户请求中收集数据，并按照单独的线程将此数据发送回审计服务器 |



#### Ranger Admin

　　Ranger admin它包含三个部分： Web页面、Rest消息处理服务以及数据库。 我们可以把它看成一个用来数据集中存储中心（所有的数据都集中存在这里，但是其它两个组件也可单独运行存在）。它的具体作用：
　　　　1. 接收UserSync进程传过来的用户、组信息。 并将它们保存到MySql数据库中。 需要注意：这些用户信息在配置权限策略的时候需要使用。这个很容易理解，就象领物料的时候，管理员需要登记物料的领用人，如果不是本公司的人，肯定就不会让你领了。
　　　　2. 提供创建policy策略的接口
　　　　3. 提供外部REST消息的处理接口



#### UserSync

　　UserSync是Ranger提供的一个用户信息同步接口，可以用来同步Linux用户信息与LDAP用户信息。通过配置项：
　　SYNC_SOURCE = LDAP/Unix
　　来确认是LDAP还是Unix， 默认情况是同步Unix信息。 

　　UserSync不是实时同步的



#### Plugin

1. NameNode启动时候，会创建一个Hdfs Plugin线程， 这个线程启动后就会读取配置信息
2. 同步完成后，hdfs plugin线程会每隔一段时间（30s）会往ranger admin发送一次消息，同步一次policy信息
3. 当用户访问的hdfs 数据的时候，就会触发权限验证，从而进行验证策略的验证



### 依赖的组件如下：

- JDK 运行RangerAdmin RangerKMS
- RDBMS 1.存储授权策略 2.存储Ranger 用户/组 3.存储审核日志
- Solr(可选) 存储审核日志
- HDFS(可选) 存储审核日志
- Kerberos(可选) 确保所有请求都被认证



