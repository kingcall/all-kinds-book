## 对比Memcache
（1）数据结构：Memcache只支持key value存储方式，Redis支持更多的数据类型，比如Key value、hash、list、set、zset；

（2）多线程：Memcache支持多线程，Redis支持单线程；CPU利用方面Memcache优于Redis；

（3）持久化：Memcache不支持持久化，Redis支持持久化；

（4）内存利用率：Memcache高，Redis低（采用压缩的情况下比Memcache高）；

（5）过期策略：Memcache过期后，不删除缓存，会导致下次取数据数据的问题，Redis有专门线程，清除缓存数据；

（6)、使用底层模型不同 它们之间底层实现方式 以及与客户端之间通信的应用协议不一样。 Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。

（7). value 值大小不同：Redis 最大可以达到 512M；memcache 只有 1mb。

（8）redis的速度比memcached快很多

（9）Redis支持数据的备份，即master-slave模式的数据备份。
