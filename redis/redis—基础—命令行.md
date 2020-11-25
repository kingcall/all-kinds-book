
## 命令行选项
```
Usage: redis-cli [OPTIONS] [cmd [arg [arg ...]]]
  -h <hostname>      Server hostname (default: 127.0.0.1).
  -p <port>          Server port (default: 6379).
  -s <socket>        Server socket (overrides hostname and port).
  -a <password>      Password to use when connecting to the server.
  -r <repeat>        Execute specified command N times.
  -i <interval>      When -r is used, waits <interval> seconds per command.
                     It is possible to specify sub-second times like -i 0.1.
  -n <db>            Database number.
  -x                 Read last argument from STDIN.
  -d <delimiter>     Multi-bulk delimiter in for raw formatting (default: \n).
  -c                 Enable cluster mode (follow -ASK and -MOVED redirections).
  --raw              Use raw formatting for replies (default when STDOUT is
                     not a tty).
  --no-raw           Force formatted output even when STDOUT is not a tty.
  --csv              Output in CSV format.
  --stat             Print rolling stats about server: mem, clients, ...
  --latency          Enter a special mode continuously sampling latency.
  --latency-history  Like --latency but tracking latency changes over time.
                     Default time interval is 15 sec. Change it using -i.
  --latency-dist     Shows latency as a spectrum, requires xterm 256 colors.
                     Default time interval is 1 sec. Change it using -i.
  --lru-test <keys>  Simulate a cache workload with an 80-20 distribution.
  --slave            Simulate a slave showing commands received from the master.
  --rdb <filename>   Transfer an RDB dump from remote server to local file.
  --pipe             Transfer raw Redis protocol from stdin to server.
  --pipe-timeout <n> In --pipe mode, abort with error if after sending all data.
                     no reply is received within <n> seconds.
                     Default timeout: 30. Use 0 to wait forever.
  --bigkeys          Sample Redis keys looking for big keys.
  --scan             List all keys using the SCAN command.
  --pattern <pat>    Useful with --scan to specify a SCAN pattern.
  --intrinsic-latency <sec> Run a test to measure intrinsic system latency.
                     The test will run for the specified amount of seconds.
  --eval <file>      Send an EVAL command using the Lua script at <file>.
  --help             Output this help and exit.
  --version          Output version and exit.
````

- -h: 指定要连接的主机

- -p: 端口

- -r: 重复执行命令

- -i: 指定时间间隔执行一次命令

- -x: 标准输入读取数据作为redis-cli的最后一个参数

- -c: 指定连接集群

- -a: 指定密码

- --scan | --pattern: 指定扫描key的模式

- --slave: 把当前客户端模拟成Redis的从节点

- --rdb: 请求redis生成RDB持久化文件，保存在本地

- --binkeys: 使用scan命令对Redis的key进行采样，从中找到内存占- 用比较大的键值

- --latency: 测试客户端到目标redis服务器的网络延迟

- --latency-dist：以图表的形式从控制台输出统计信息

- --stat实时获取Redis的重要统计信息

- --raw 和 --no-raw: 是否以原始格式化结果返回

  

Examples:

```
  cat /etc/passwd | redis-cli -x set mypasswd
  redis-cli get mypasswd
  redis-cli -r 100 lpush mylist x
  redis-cli -r 100 -i 1 info | grep used_memory_human:
  redis-cli --eval myscript.lua key1 key2 , arg1 arg2 arg3
  redis-cli --scan --pattern '*:12345*'

  (Note: when using --eval the comma separates KEYS[] from ARGV[] items)
```

### 运维命令
- redis-server	
- redis-cli	
- redis-benchmark	
- redis-check-aof	
- redis-check-dump	
- redis-sentinel	

### 常见命令

```
• Save	SAVE 命令用于创建当前数据库的备份。redis 安装目录中创建dump.rdb文件。
	如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可   CONFIG GET dir 获取安装，目录
Bgsave	

Client list	
DBSIZE	返回当前数据库的 key 的数量， info keyspace
FLUSHALL	删除所有数据库的所有key
FLUSHDB 	删除当前数据库的所有key
ROLE 	返回主从实例所属的角色
CONFIG get requirepass	验证有没有设置密码
redis-benchmark	 同时执行 10000 个请求来检测性能   
	这个命令有很多参数，而且不是运行在redis 的shell 里面的 
config get maxclients	最大连接数是被直接硬编码在代码里面的,可以通过配置文件或命令修改
	redis-server --maxclients 100000  启动的时候修改
CLIENT SETNAME	设置当前连接的名称
Keys 	*: 通配任意多个字符?: 通配单个字符     []: 通配括号内的某1个字符 
scan	Keys 命令的升级版本  分多次遍历数据库，SCAN命令是一个基于游标的迭代器。这意味着命令每次被调用都需要使用上一
	次这个调用返回的游标作为该次调用的游标参数，以此来延续之前的迭代过程
	如果直接匹配的话，默认每次返回10条数据
	scan 0 match vote*   会返回当前页中匹配上的数据
	
info 	info keyspace/clients/Memory/
exist	exist s1 键存在返回1,不存在返回0
type 	type s1  查看键的数据类型，不存在则返回  none
ttl 	ttl s1 返回键的过期时间，整数  还有多长时间过期，-1 没有设置过期时间   -2 键不存在
OBJECT encoding  	OBJECT encoding s1 返回键的内部编码实现
select 	切换数据库
config rewrite	将 redis 的配置信息持久化到配置文件

```

## 例子
### 批量删除
redis-cli -a kingcall keys "*" |xargs redis-cli -a kingcall del

redis-cli -h localhost -p 6379 -a kingcall set kingcall kingcall

redis-cli -a kingcall keys "*"

### 显示中文
![image-20201125184739067](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/18:47:39-image-20201125184739067.png)
### 同步hive 维表到redis

### 数据导出
- redis-cli -h 10.52.7.100 -p 8001 --csv  hgetall spark:job:list > test
- redis-cli -h 10.52.7.100 -p 8001 --raw  hgetall spark:job:list > test(结果可读性更好)

### 产生测试数据

- DEBUG populate 300000 ioock 
- debug populate 10000

### 从标准选项获取输入

- echo "set k1 v1" | redis-cli -x
- echo "set" | redis-cli -x set k2



### pipeline 的使用

- 在命令行客户端中使用参数 --pipe
- 高级语言的大多数客户端都提供了pipeline机制

