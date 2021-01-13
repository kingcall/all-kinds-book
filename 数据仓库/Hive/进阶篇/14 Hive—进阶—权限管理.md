[TOC]



## Hive 权限管理

权限管理是为了验证某个用户是否有执行某项操作的权限，安全认证是为了认证用户是合法用户，今天我们主要介绍的是权限管理而不是安全认证



集群安全下需求：

- 支持多组件，最好能支持当前大数据技术栈的主要组件，HDFS、HBASE、HIVE、 YARN、KAFKA等
- 支持细粒度的权限控制，可以达到HIVE列，HDFS目录，HBASE列,YARN队列
- 开源，社区活跃，按照现有的集群情况改动尽可能的小，而且要符合业界的趋势。



现有方案：

- Hadoop、Hive本 身的权限控制
- Kerberos安全认证
- Apache Ranger权限管理方案

![image-20210113171545935](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210113171545935.png)

### Hadoop 和 Hive 自带的权限认证

Hadoop权限：

- Hadoop分布式文件系统实现了一个和POSIX系统类似的文件和目录的权限模型
- 每个文件和目录有一个所有者（owner）和一个组（group）
- 文件或目录对其所有者、同组的其他用户以及所有其他用户分别有着不同的权限
- 文件或目录操作都传递路径名给NameNode，对路径做权限检查
- 启动NameNode的用户是超级用户，能够通过所有的权限检查
- 通过配置可以指定一组特定的用户为超级用户

Hive可以利用Hadoop的文件系统进行权限管理，实现原理就是判断当前用户是否有操作文件的权限

Hive默认的权限控制并不是完全安全的。hive的权限控制是为了防止用户不小心做了不合适的操作。

 

### **三种授权模型**

**1、Storage Based Authorization in the Metastore Server**
基于存储的授权 - 可以对Metastore中的元数据进行保护，但是没有提供更加细粒度的访问控制（例如：列级别、行级别）。
**2、SQL Standards Based Authorization in HiveServer2**
基于SQL标准的Hive授权 - 完全兼容SQL的授权模型，推荐使用该模式。基于 SQL 标准的完全兼容 SQL 的授权模型，除支持对于用户的授权认证，还支持角色 role 的授权认证

role 可理解为是一组权限的集合，通过 role 为用户授权一个用户可以具有一个或多个角色,默认包含两种角色：public、admin

通过hiveserver2的方式访问hive数据,默认提供两种角色：public和admin，所有用户默认属于角色public，而授权则必须是具有角色admin的用户才可以完成（普通用户仅可以将自己获得的权限授权给其它用户）

因此我们必须添加至少一个用户拥有角色admin，创建/删除角色命令用户和组使用的是Linux机器上的用户和组，而角色必须自己(hive)创建。

public 用户有执行授权操作的权限,但是默认情况下public 用户是没有创建表的权限的

**3、Default Hive Authorization (Legacy Mode)**
hive默认授权 - 设计目的仅仅只是为了防止用户产生误操作，而不是防止恶意用户访问未经授权的数据



#### Default Hive Authorization (Legacy Mode)

Hive默认的权限控制并不是完全安全的。hive的权限控制是为了防止用户不小心做了不合适的操作。而不是组织非法用户访问数据

这是因为这种权限管理机制是不完善的，它并没有权限校验的机制，例如你执行`grant` 操作，它并不去检测你是否有权限，它的检测发生在SQL编译的阶段。

#### Storage Based Authorization in the Metastore Server

Hive早期版本是通过Linux的用户和用户组来控制用户权限，无法对Hive表的CREATE、SELECT、DROP等操作进行控制。现Hive基于元数据库来管理多用户并控制权限。数据分为元数据和数据文件两部分，元数据存储在mysql，而数据文件则是HDFS，控制元数据即控制可访问的数据文件。

通过Hcatcalog API访问hive数据的方式，实际是通过访问metastore元数据的形式访问hive数据，这类有MapReduce，Impala，Pig，Spark SQL，Hive Command line等方式，其实就是说这个权限控制发生在和Metastore服务交互的时候，实现方式就是在调用Metastore Api 的时候实现权限校验，主要是防止恶意用户对Metastore数据的访问和修改，但是没有提供更加细粒度的访问控制（例如：列级别、行级别）。

下面是其配置

```xml
<property>
  <name>hive.security.metastore.authorization.manager</name>
  <value>org.apache.hadoop.hive.ql.security.authorization.DefaultHiveMetastoreAuthorizationProvider</value>
  <description>authorization manager class name to be used in the metastore for authorization.
  The user defined authorization class should implement interface
  org.apache.hadoop.hive.ql.security.authorization.HiveMetastoreAuthorizationProvider.
  </description>
 </property>
 
<property>
  <name>hive.security.metastore.authenticator.manager</name>
  <value>org.apache.hadoop.hive.ql.security.HadoopDefaultMetastoreAuthenticator</value>
  <description>authenticator manager class name to be used in the metastore for authentication.
  The user defined authenticator should implement interface 
  org.apache.hadoop.hive.ql.security.HiveAuthenticationProvider.
  </description>
</property>
 
<property>
  <name>hive.metastore.pre.event.listeners</name>
  <value> </value>
  <description>pre-event listener classes to be loaded on the metastore side to run code
  whenever databases, tables, and partitions are created, altered, or dropped.
  Set to org.apache.hadoop.hive.ql.security.authorization.AuthorizationPreEventListener
  if metastore-side authorization is desired.
  </description>
</property>
```



#### SQL Standards Based Authorization in HiveServer2

- 完全兼容SQL的授权模型
- 除支持对于用户的授权认证，还支持角色role的授权认证
- role可理解为是一组权限的集合，通过role为用户授权
- 一个用户可以具有一个或多个角色

**默认包含另种角色：public、admin** 其中admin 用户可以在配置文件里进行配置，角色的命名是大小写不敏感的，这一点和SQL相同，但是用户名是的大小写敏感的。

```
<property>
  <name>hive.users.in.admin.role</name>
  <value>root</value>
</property>
```

- 启用当前认证方式之后，dfs, add, delete, compile, and reset 等命令被禁用。通过 set 命令设置 hive configuration 的方式被限制某些用户使用。也可通过修改配置文件 hive-site.xml 中hive.security.authorization.sqlstd.confwhitelist 进行配置，哪些用户可以使用这些命令，其实就是白名单（add 或者 drop 函数的命令属于admin 角色所有，所以这个时候如果要添加自定义函数的话可以通过admin 用户添加一个永久函数然后其他用户使用的方式来完成）
- 添加、删除函数以及宏（批量规模）的操作，仅为具有 admin 的用户开放。
- 用户自定义函数（开放支持永久的自定义函数），可通过具有 admin 角色的
- Transform功能被禁用。


**public 角色**

默认情况下所有用户默认属于角色public，而授权则必须是具有角色admin的用户才可以完成（普通用户仅可以将自己获得的权限授权给其它用户）

public 用户有执行授权操作的权限,但是默认情况下public 用户是没有创建表的权限的

**admin 角色**

用户除了admin 角色的其他角色都是默认给用户的，也就是说只要你有这个角色的权限，当你执行`show current roles;`都是可以看到的，但是admin角色不行，即使你属于这个角色的列表，你也需要通过`set role admin;` 来获取这个角色的权限

![image-20210113094349584](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210113094349584.png)

也就是说admin 角色是不在用户的`current roles` 列表里的

可以通过`set hive.users.in.admin.role;`查看哪些用户拥有admin 权限

**基本配置**

```sql
<!-- 开启身份验证 默认是没有开启的 -->
<property>
  <name>hive.security.authorization.enabled</name>
  <value>true</value>
</property>
<property>
  <name>hive.server2.enable.doAs</name>
  <value>false</value>
</property>
<!-- admin 角色的用户列表  -->
<property>
  <name>hive.users.in.admin.role</name>
  <value>root</value>
</property>
<property>
  <name>hive.security.authorization.manager</name>
  <value>org.apache.hadoop.hive.ql.security.authorization.plugin.sqlstd.SQLStdHiveAuthorizerFactory</value>
</property>
<property>
  <name>hive.security.authenticator.manager</name>
  <value>org.apache.hadoop.hive.ql.security.SessionStateUserAuthenticator</value>
</property>
```

注意：**拥有admin角色的用户需要运行命令“set role admin"去获取admin角色的权限**，也会就是说即使你在admin 角色列表里，依然需要去获取一次才能拥有权限

**HIVE新建文件权限**

Hive由一个默认的设置来配置新建文件的默认权限。创建文件授权掩码为0002，即664权限，具体要看hadoop与hive用户配置。

```xml
<property>  
  <name>hive.files.umask.value</name>  
  <value>0002</value>  
  <description>The dfs.umask value for the hive created folders</description>  
</property> 
```



**HIVE授权存储检查**

当hive.metastore.authorization.storage.checks属性被设置成true时，Hive将会阻止没有权限的用户进行表删除操作。不过这个配置的默认值是false，应该设置成true。

```xml
<property>  
  <name>hive.metastore.authorization.storage.checks</name>  
  <value>true</value>  
  <description>Should the metastore do authorization checks against  
  the underlying storage for operations like drop-partition (disallow  
  the drop-partition if the user in question doesn't have permissions  
  to delete the corresponding directory on the storage).</description>  
</property>
```

**表创建者拥有表的全部权限**

```xml
<property>  
  <name>hive.security.authorization.createtable.owner.grants</name>  
  <value>ALL</value>  
  <description>The privileges automatically granted to the owner whenever a table gets created.
    An example like "select,drop" will grant select and drop privilege to the owner of the table
  </description>  
</property>  
```

 这个配置默认是NULL，建议将其设置成ALL，**让用户能够访问自己创建的表，否则表的创建者无法访问该表，这明显是不合理的**。 

hive 中表是那个用户创建的，那个用户就是Hive 表的Owner 其实就是HDFS 上文件夹的Owner ，这一点和Linux 的操作系统权限是一样的，我用不同的用户登陆进入Hive 终端创建出来的表的Owner所属关系如下图

![image-20201230103332044](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230103332044-20210112203945713.png)



### Hive 的权限操作

在开始之前我们先说一点关于HiveServer2的命令行客户端的一个需要注意的事情,就是当你在不直接指定当前用户的时候登陆上去之后，此时Hive 的用户并不你系统的当前用户，而是一个匿名用户，我因为这个事情头疼了好一会的，所以开始之前把这个单独说一下

![image-20210112205134932](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210112205134932.png)



#### 1、授权给用户

首先我创建了一个新的用户'kingcall',然后切到这个用户上来，然后用这个用户连接到hive 上来`beeline -u jdbc:hive2://localhost:10000/ods -u root  -p www1234 -n kingcall`，接下我们用admin 权限的用户创建一个表

`create table role_test(id int,name string);` 然后让kingcall 用户去查询

```
0: jdbc:hive2://localhost:10000/ods> select * from role_test;
Error: Error while compiling statement: FAILED: HiveAccessControlException Permission denied: Principal [name=kingcall, type=USER] does not have following privileges for operation QUERY [[SELECT] on Object [type=TABLE_OR_VIEW, name=ods.role_test]] (state=42000,code=40000)
```

接下来我们就需要给这个用户授权了，我们上面配置了表的创建者拥有表的全部权限，这个时候表的创建者就可以完成赋权操作(不需要admin角色)

`GRANT SELECT ON table role_test to user kingcall;` 这下我们让kingcall 用户再去查询，就可以了，这就是基本赋权操作

![image-20210112211105878](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210112211105878.png)

如果这个时候你去执行delete 操作的话，依然是不可执行的，因为上面赋权的时候我们仅仅给了SELECT 权限，这个时候你可以看当前用户的权限` SHOW GRANT USER KINGCAL;`

```
Error: Error while processing statement: FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. Error showing privileges:
User : liuwenqiang is not allowed check privileges of another user : KINGCAL. User has to belong to ADMIN role and have it as current role, for this action. (state=08S01,code=1)
```

虽然我们的liuwenqiang用户是admin 角色列表里的用户，但是由于当前没有获取admin 角色，所以它是不能查看其它用户的权限的,所以需要先执行`set role admin;`才能查看其它用户的权限

![image-20210112212833177](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210112212833177.png)

其实这里我们就简单演示了如何为用户赋权，其实再任何系统里我们永远保持最小权限的原则，所以建议赋权的时候也采取这样的策略，可以给用户的权限有很多，这里就不一一演示，下面我们有一个表里面列出了Hive 中所有可以操作的权限,除了赋权用户某一个表的权限，我们也可赋权整个库的权限给用户，当然也可以收回权限。



#### 2、授权给组

当Hive里面用于N多用户和N多张表的时候，管理员给每个用户授权每张表会让他崩溃的。所以，这个时候就可以进行组(GROUP)授权。Hive里的用户组的定义等价于POSIX里面的用户组。

```sql
-- 基于数据库
grant select on database default to  group  admin;
-- 基于某张表
grant select on table ppdata  to  group  admin;
```



#### 3、角色管理

当给用户组授权变得不够灵活的时候，角色(ROLES)就派上用途了。用户可以被放在某个角色之中，然后角色可以被授权。角色不同于用户组是由操作系统控制的，它是由Hive内部进行管理的。而用户和用户组是操作系统管理的。

**创建角色**

` CREATE ROLE role_name;`

创建角色需要admin 权限，并且不能使用ALL, DEFAULT , NONE 保留字，建议创建角色的时候应该见名知意`role_database_select` 也就是这个用户有表的查询操作权限，其实一般情况下我们让角色的权限尽量原子化，然后通过配合用户组完成整个权限的控制,例如我们可以将我们的报表层库的查询权限创建一个角色，这个角色就可以业务方的数据分析人员进行取数，或者我们可以将报表层库的权限可以在按照业务线进行细分。

**给角色赋权**

```sql
CREATE ROLE role_ods_select;
--赋予role_name拥有在database_name创建的权限,grant select on [table] table_name to role role_name 也可以将表的权限赋给某个角色
grant select on database ods to role role_ods_select;
 -- 查询某个角色拥有的权限
 show grant role role_ods_select on database ods;
 +-----------+--------+------------+---------+------------------+-----------------+------------+---------------+----------------+--------------+
| database  | table  | partition  | column  |  principal_name  | principal_type  | privilege  | grant_option  |   grant_time   |   grantor    |
+-----------+--------+------------+---------+------------------+-----------------+------------+---------------+----------------+--------------+
| ods       |        |            |         | role_ods_select  | ROLE            | SELECT     | false         | 1610504539000  | liuwenqiang  |
+-----------+--------+------------+---------+------------------+-----------------+------------+---------------+----------------+--------------+
```



**将角色赋给用户**

```sql
-- 我们将我们前面演示的时候给kingcall用户的role_test表的select权限收回
revoke select on table role_test from user kingcall;
-- 然后你再用kingcall去执行select from role_test 就没有权限了，然后我们将 role_ods_select 角色给用户
 grant role role_ods_select to user kingcall;
 -- 可以查看当前用户都有哪些角色，注意如果你使用show current roles 获取到底依然是之前的角色，你需要使用set role 之后才能看到新角色
show role grant user kingcall;
+------------------+---------------+----------------+--------------+
|       role       | grant_option  |   grant_time   |   grantor    |
+------------------+---------------+----------------+--------------+
| public           | false         | 0              |              |
| role_ods_select  | false         | 1610504936000  | liuwenqiang  |
+------------------+---------------+----------------+--------------+
 -- 然后再去执行查询操作
 select * from role_test;
```

显示用户的当前角色列表`SHOW CURRENT ROLES`  所有用户都可以执行

`show roles` 显示hive 当前全部的角色，这个命令admin 才能执行

`show role grant user kingcall;` 普通只能查看自己的，admin 用户可以查看全部人的

`SHOW PRINCIPALS role_ods_create` 可以查看一个角色的赋予了哪些用户

` show grant user kingcall on all;`

`show grant role  role_ods_select  on all;`

` show grant on table test_role;`



**set role** 

set role 是个有意思的命令，它的本意是获取用户的权限，因为角色是管理员分给用户的，已经分了，所以你不能认为是获取角色，因为用户的当前角色

```sql
CREATE ROLE role_ods_create;
grant ALL on database ods to role role_ods_create;
-- 我们查看一下这个角色现在都有哪些权限
 show grant role role_ods_create on database ods;
+-----------+--------+------------+---------+------------------+-----------------+------------+---------------+----------------+--------------+
| database  | table  | partition  | column  |  principal_name  | principal_type  | privilege  | grant_option  |   grant_time   |   grantor    |
+-----------+--------+------------+---------+------------------+-----------------+------------+---------------+----------------+--------------+
| ods       |        |            |         | role_ods_create  | ROLE            | DELETE     | false         | 1610506484000  | liuwenqiang  |
| ods       |        |            |         | role_ods_create  | ROLE            | INSERT     | false         | 1610506484000  | liuwenqiang  |
| ods       |        |            |         | role_ods_create  | ROLE            | SELECT     | false         | 1610506484000  | liuwenqiang  |
| ods       |        |            |         | role_ods_create  | ROLE            | UPDATE     | false         | 1610506484000  | liuwenqiang  |
+-----------+--------+------------+---------+------------------+-----------------+------------+---------------+----------------+--------------+
-- 然后你通过show current roles 就会发现Kingcall 用户并没有这个权限
```

SET ROLE (role_name|ALL|NONE);





#### 4、Hive权限管理命令

set role admin;  # 将当前用户角色设置为admin

```sql
--角色的添加、删除、查看、设置：
CREATE ROLE role_name;  
-- 删除角色
DROP ROLE role_name; 
-- 设置角色(给当前用户设置角色)
SET ROLE (role_name|ALL|NONE); 
-- 查看当前具有的角色
SHOW CURRENT ROLES;  
SHOW ROLE GRANT USER liuwenqiang;
-- 查看所有存在的角色
SHOW ROLES;  
-- 查看用户的权限
SHOW GRANT USER root ON DATABASE ods;
-- 查看用户的角色
show role grant user user_name;
-- 回收某个角色的某个权限：
revoke create on database database_name from role role_name
revoke select on [table] table_name from role role_name
-- 查看某个角色在某张表或某个数据库的权限：
show grant [role|user] role_name on database database_name
show grant [role|user] role_name on [table] table_name
```

#### 5、Hive权限分配表

| Action                                          | Select       | Insert     | Update | Delete            | Owership        | Admin | URL Privilege(RWX Permission + Ownership)    |
| :---------------------------------------------- | :----------- | :--------- | :----- | :---------------- | :-------------- | :---- | :------------------------------------------- |
| ALTER DATABASE                                  |              |            |        |                   |                 | Y     |                                              |
| ALTER INDEX PROPERTIES                          |              |            |        |                   | Y               |       |                                              |
| ALTER INDEX REBUILD                             |              |            |        |                   | Y               |       |                                              |
| ALTER PARTITION LOCATION                        |              |            |        |                   | Y               |       | Y (for new partition location)               |
| ALTER TABLE (all of them except the ones above) |              |            |        |                   | Y               |       |                                              |
| ALTER TABLE ADD PARTITION                       |              | Y          |        |                   |                 |       | Y (for partition location)                   |
| ALTER TABLE DROP PARTITION                      |              |            |        | Y                 |                 |       |                                              |
| ALTER TABLE LOCATION                            |              |            |        |                   | Y               |       | Y (for new location)                         |
| ALTER VIEW PROPERTIES                           |              |            |        |                   | Y               |       |                                              |
| ALTER VIEW RENAME                               |              |            |        |                   | Y               |       |                                              |
| ANALYZE TABLE                                   | Y            | Y          |        |                   |                 |       |                                              |
| CREATE DATABASE                                 |              |            |        |                   |                 |       | Y (if custom location specified)             |
| CREATE FUNCTION                                 |              |            |        |                   |                 | Y     |                                              |
| CREATE INDEX                                    |              |            |        |                   | Y (of table)    |       |                                              |
| CREATE MACRO                                    |              |            |        |                   |                 | Y     |                                              |
| CREATE TABLE                                    |              |            |        |                   | Y (of database) |       | Y (for create external table – the location) |
| CREATE TABLE AS SELECT                          | Y (of input) |            |        |                   | Y (of database) |       |                                              |
| CREATE VIEW                                     | Y + G        |            |        |                   |                 |       |                                              |
| DELETE                                          |              |            |        | Y                 |                 |       |                                              |
| DESCRIBE TABLE                                  | Y            |            |        |                   |                 |       |                                              |
| DROP DATABASE                                   |              |            |        |                   | Y               |       |                                              |
| DROP FUNCTION                                   |              |            |        |                   |                 | Y     |                                              |
| DROP INDEX                                      |              |            |        |                   | Y               |       |                                              |
| DROP MACRO                                      |              |            |        |                   |                 | Y     |                                              |
| DROP TABLE                                      |              |            |        |                   | Y               |       |                                              |
| DROP VIEW                                       |              |            |        |                   | Y               |       |                                              |
| DROP VIEW PROPERTIES                            |              |            |        |                   | Y               |       |                                              |
| EXPLAIN                                         | Y            |            |        |                   |                 |       |                                              |
| INSERT                                          |              | Y          |        | Y (for OVERWRITE) |                 |       |                                              |
| LOAD                                            |              | Y (output) |        | Y (output)        |                 |       | Y (input location)                           |
| MSCK (metastore check)                          |              |            |        |                   |                 | Y     |                                              |
| SELECT                                          | Y            |            |        |                   |                 |       |                                              |
| SHOW COLUMNS                                    | Y            |            |        |                   |                 |       |                                              |
| SHOW CREATE TABLE                               | Y+G          |            |        |                   |                 |       |                                              |
| SHOW PARTITIONS                                 | Y            |            |        |                   |                 |       |                                              |
| SHOW TABLE PROPERTIES                           | Y            |            |        |                   |                 |       |                                              |
| SHOW TABLE STATUS                               | Y            |            |        |                   |                 |       |                                              |
| TRUNCATE TABLE                                  |              |            |        |                   | Y               |       |                                              |
| UPDATE                                          |              |            | Y      |                   |                 |       |                                              |

   ALL:所有权限

   ALTER:允许修改元数据（modify metadatadata of object）---表信息数据

   UPDATE:允许修改物理数据（modify physicaldata of object）---实际数据

   CREATE:允许进行Create操作

   DROP:允许进行DROP操作

   LOCK:当出现并发的使用允许用户进行LOCK和UNLOCK操作

   SELECT:允许用户进行SELECT操作：

   SHOW_DATABASE:允许用户查看可用的数据库



### 扩展

#### 如何确定一个SQL的执行需要哪些权限呢

前面我们介绍了，如何为用户赋权，但是当我们执行一个比较复杂的SQL的时候我们怎么知道这个SQL都需要哪些权限呢。还记得我们学习执行计划的时候有一个选项参数吗？没错，就是它。

 EXPLAIN AUTHORIZATION  select * from role_test;

```
|   hdfs://kingcall:9000/tmp/hive/liuwenqiang/5b3a7a3a-fb84-442e-a91b-855ed826b9ef/hive_2021-01-12_21-58-26_399_5494553353081655039-5/-mr-10001 |
| CURRENT_USER:                                      |
|   kingcall                                         |
| OPERATION:                                         |
|   QUERY
```



#### 超级管理员的实现

前面我们也提到了，Hive中没有超级管理员，任何用户都可以进行Grant/Revoke操作(表的创建这或者库的创建者)，这使得权限管理失去意义。为了解决这个问题，就需要我们开发实现自己的权限控制类，确保某个用户为超级用户。

需要引入依赖

```
<dependencies>
    <dependency>
        <groupId>org.apache.hive</groupId>
        <artifactId>hive-exec</artifactId>
           <version>3.1.0</version>
    </dependency>
</dependencies>
```

接下来实现自定义的hook

```java
package com.kingcall.bigdata.HiveAccess;

import com.google.common.base.Joiner;
import org.apache.hadoop.hive.ql.parse.*;
import org.apache.hadoop.hive.ql.session.SessionState;

/**
 * 自定义Hive的超级用户
 *
 * @author 01
 * @date 2020-11-09
 **/
public class HiveAdmin extends AbstractSemanticAnalyzerHook {

    /**
     * 定义超级用户，可以定义多个
     */
    private static final String[] ADMINS = {"root"};

    /**
     * 权限类型列表
     */
    private static final int[] TOKEN_TYPES = {
            HiveParser.TOK_CREATEDATABASE, HiveParser.TOK_DROPDATABASE,
            HiveParser.TOK_CREATEROLE, HiveParser.TOK_DROPROLE,
            HiveParser.TOK_GRANT, HiveParser.TOK_REVOKE,
            HiveParser.TOK_GRANT_ROLE, HiveParser.TOK_REVOKE_ROLE,
            HiveParser.TOK_CREATETABLE
    };

    /**
     * 获取当前登录的用户名
     *
     * @return 用户名
     */
    private String getUserName() {
        boolean hasUserName = SessionState.get() != null &&
                SessionState.get().getAuthenticator().getUserName() != null;

        return hasUserName ? SessionState.get().getAuthenticator().getUserName() : null;
    }

    private boolean isInTokenTypes(int type) {
        for (int tokenType : TOKEN_TYPES) {
            if (tokenType == type) {
                return true;
            }
        }

        return false;
    }

    private boolean isAdmin(String userName) {
        for (String admin : ADMINS) {
            if (admin.equalsIgnoreCase(userName)) {
                return true;
            }
        }

        return false;
    }

    @Override
    public ASTNode preAnalyze(HiveSemanticAnalyzerHookContext context, ASTNode ast) throws SemanticException {
        if (!isInTokenTypes(ast.getToken().getType())) {
            return ast;
        }

        String userName = getUserName();
        if (isAdmin(userName)) {
            return ast;
        }

        throw new SemanticException(userName +
                " is not Admin, except " +
                Joiner.on(",").join(ADMINS)
        );
    }
}

```

打包添加到hive 的lib 目录下`cp target/original-HiveUDF-0.0.4.jar /usr/local/hive-3.1.2/lib/`

```sql
<property>
    <name>hive.semantic.analyzer.hook</name>
    <value>com.kingcall.bigdata.HiveAccess.HiveAdmin</value>
    <description>使用钩子程序，识别超级管理员，进行授权控制</description>
</property>
```

重启hiveserver2 服务

![image-20210113115329977](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210113115329977.png)

然后你可以尝试进行赋权操作

```sql
grant select on table role_test to user kingcall;
```

然后你就得到了下面的错误，这样我们就达到控制权限的目的了

`Error: Error while compiling statement: FAILED: SemanticException hive is not Admin, except root (state=42000,code=40000)`

## 总结

1. 我们可以通过视图的方式达到更喜欢的权限管理，例如实现字段级别的权限控制
2. 对象(表、视图、databases) 的所有权一般是归创建者所有的，包括执行授权的权限
3. admin 用户可以在配置文件里进行配置，角色的命名是大小写不敏感的，这一点和SQL相同，但是用户名是的大小写敏感的



参考文章:https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+Based+Hive+Authorization

