[TOC]



## 权限认证

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpeGlua3VhbjMyOA==,size_16,color_FFFFFF,t_70.png)

### Hadoop 和 Hive 自带的权限认证

Hadoop 实现了基于文件和目录的权限认证，和linux 类似，对文件的操作，都会将路径名称传递个NameNode 做权限验证，启动NameNode 的用户是超级用户，能通过所有的权限检查，可以配置哪些用户属于超级用户组

Hive 可以利用Hadoop的文件系统进行权限管理，当然Hive 也实现了基于元数据的权限管理，有点类似数据库的权限，不同的是Hive 在数据库权限的基础上抽象出了RBAC 的模型，也就是综合了二者实现了基于元数据的RBAC模型

 HIVE授权管理，类似于操作系统权限可以授予给不同的主题，如用户(USER)，组(GROUP)，角色(ROLES)，Hive还是支持相当多的权限管理功能，满足一般数据仓库的使用，同时HIVE能支持自定义权限。但是需要注意的是Hive 中没有超级管理员任何用户都可以执行授权操作和撤销授权的操作，这就导致了很多不安全的因素

Hive的权限控制并不是完全安全的。hive的权限控制是为了防止用户不小心做了不合适的操作。



### **三种授权模型**

**1、Storage Based Authorization in the Metastore Server**
基于存储的授权 - 可以对Metastore中的元数据进行保护，但是没有提供更加细粒度的访问控制（例如：列级别、行级别）。
**2、SQL Standards Based Authorization in HiveServer2**
基于SQL标准的Hive授权 - 完全兼容SQL的授权模型，推荐使用该模式。
**3、Default Hive Authorization (Legacy Mode)**
hive默认授权 - 设计目的仅仅只是为了防止用户产生误操作，而不是防止恶意用户访问未经授权的数据



### Hive - SQL Standards Based Authorization in HiveServer2

- 完全兼容SQL的授权模型
- 除支持对于用户的授权认证，还支持角色role的授权认证
- role可理解为是一组权限的集合，通过role为用户授权
- 一个用户可以具有一个或多个角色

**默认包含另种角色：public、admin**

其中admin 用户可以在配置文件里进行配置

```
<property>
  <name>hive.users.in.admin.role</name>
  <value>root</value>
</property>
```



#### HIVE新建文件权限

Hive由一个默认的设置来配置新建文件的默认权限。创建文件授权掩码为0002，即664权限，具体要看hadoop与hive用户配置。

```xml
<property>  
  <name>hive.files.umask.value</name>  
  <value>0002</value>  
  <description>The dfs.umask value for the hive created folders</description>  
</property> 
```



#### HIVE授权存储检查

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



![image-20201230104625729](/Users/liuwenqiang/Library/Application%20Support/typora-user-images/image-20201230104625729.png)

#### HIVE身份验证

开启Hive的身份认证功能，默认是false

```xml
<property>  
  <name>hive.security.authorization.enabled</name>   
  <value>true</value>  
  <description>Enable or disable the hive client authorization</description>  
</property>  
```

表创建者用于的权限配置项

```xml
<property>  
  <name>hive.security.authorization.createtable.owner.grants</name>  
  <value>ALL</value>  
  <description>The privileges automatically granted to the owner whenever  
  a table gets created.An example like "select,drop" will grant select  
  and drop privilege to the owner of the table</description>  
</property>  
```

 这个配置默认是NULL，建议将其设置成ALL，让用户能够访问自己创建的表。

hive 中表是那个用户创建的，那个用户就是Hive 表的Owner 其实就是HDFS 上文件夹的Owner ，这一点和Linux 的操作系统权限是一样的，我用不同的用户登陆进入Hive 终端创建出来的表的Owner所属关系如下图

![image-20201230103332044](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230103332044.png)



### 演示

首先我们尝试在命令行里开启权限检测，发现失败了(我下面的用户都是root 用户，hive 的root 用户)

![image-20201230102200884](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230102200884.png)

```
Error: Error while processing statement: Cannot modify hive.security.authorization.enabled at runtime. It is in the list of parameters that can't be modified at runtime or is prefixed by a restricted variable (state=42000,code=1)
```

于是我去配置文件中开启授权检测,然后直接建表发现还是可以成功，那就说明当前登陆用户是有建表权限的，那我就看一下当前用户拥有什么权限

```
 SHOW GRANT USER root ON DATABASE ods;
```

![image-20201230105818960](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230105818960.png)

然后我们发现这个用户并没有什么单独的授权，这是为什呢，那我们看一下当前用户有什么角色吗，是不是这个角色导致有创建表的权限

![image-20201230105810322](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230105810322.png)

```
show grant role public on database ods;
```

![image-20201230110208251](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201230110208251.png)

还是没很么发现，这是为什么,所以这个时候我们没有办法发现root 用户都有什么样的权限，而且当我们尝试创建角色或者尝查看其它用户的角色的时候发现是不能查看的

```
Error: Error while processing statement: FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. 
Error showing privileges: User : root is not allowed check privileges of another user : liuwenqiang. 
User has to belong to ADMIN role and have it as current role, for this action. (state=08S01,code=1)
```

既然如此，我们尝试一下将admin 角色赋给当前用户(root)





## 1、授权给用户

现在通过授权方式，将权限授予给当前用户：

```sql
hive> set system:user.name;  
system:user.name=hadoop  
hive> GRANT CREATE ON DATABASE default TO USER hadoop;  
hive> CREATE TABLE auth_test (key INT, value STRING);  

通过SHOW GRANT命令确认我们拥有的权限:



hive> SHOW GRANT USER hadoop ON DATABASE default;    



database default    



principalName hadoop    



principalType USER    



privilege Create    



grantTime Sun May 06 17:18:10 EDT 2018    



grantor hadoop  
```



## 2、授权给组

当Hive里面用于N多用户和N多张表的时候，管理员给每个用户授权每张表会让他崩溃的。
所以，这个时候就可以进行组(GROUP)授权。
Hive里的用户组的定义等价于POSIX里面的用户组。

```sql
hive> CREATE TABLE auth_test_group(a int,b int);  



 



hive> SELECT * FROM auth_test_group; 



 



Authorization failed:No privilege 'Select' found for inputs  



{ database:default, table:auth_test_group, columnName:a}.  



Use show grant to get more details.  







hive> GRANT SELECT on table auth_test_group to group hadoop;  







hive> SELECT * FROM auth_test_group;  



OK  



Time taken: 0.119 seconds 
```

## 3、角色管理

当给用户组授权变得不够灵活的时候，角色(ROLES)就派上用途了。用户可以被放在某个角色之中，然后角色可以被授权。角色不同于用户组，是由Hadoop控制的，它是由Hive内部进行管理的。



```sql
hive> CREATE TABLE auth_test_role (a int , b int);  
hive> SELECT * FROM auth_test_role;  

Authorization failed:No privilege 'Select' found for inputs  



{ database:default, table:auth_test_role, columnName:a}.  



Use show grant to get more details.  

hive> CREATE ROLE users_who_can_select_auth_test_role;  

hive> GRANT ROLE users_who_can_select_auth_test_role TO USER hadoop;  

hive> GRANT SELECT ON TABLE auth_test_role  

> TO ROLE users_who_can_select_auth_test_role;  

hive> SELECT * FROM auth_test_role;  


```

显示用户当前角色列表

```sql
SHOW CURRENT ROLES;
```



DROP ROLE role_name;	

CREATE ROLE create_r;

CREATE ROLE create_r;
Error: Error while processing statement: FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. Current user : root is not allowed to add roles. User has to belong to ADMIN role and have it as current role, for this action. (state=08S01,code=1)



### Kerberos 方案



### Ranger 方案

apache 开源的方案

