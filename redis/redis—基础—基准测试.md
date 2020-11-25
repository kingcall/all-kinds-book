## redis-benchmark
- Redis性能测试工具
- -c: 代表客户端的并发数量,默认50
- -n <requests> 代表客户端总的请求量，默认是100000
- -q: 仅仅显示request per second信息
- -r: 向redis插入更多随机的key
- -P: 代表没一个请求pipeline的数据量
- -k 1|0: 是否使用keepalive
- -t: 对指定命令进行基准测试

### 例子
- redis-benchmark -c 100 -n 20000表示100个客户端同时请求redis，一共执行20000次
-  redis-benchmark -c 100 -n 20000 -r 10000(默认执行完只有3个key)
- redis-benchmark -t set,get (测试特定的命令)



![image-20201125212241667](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:22:42-image-20201125212241667.png)

## redis-server
- --test-memory: 检测当前操作系统能否稳定的分配指定容量的内存给Redis，通过这种检测可以有效避免因为内存问题造成Redis崩溃
- redis-server –test-memory 1024 检测操作系统能否提供1G内存给Redis