## 单节点

- 解压 进入目录 执行 make,成功后进入src 执行 make install（ make PREFIX=/usr/local/redis install 或者直接安装到特定目录）
> 会帮你自动创建bin目录

![image-20201125211717286](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:17:17-image-20201125211717286.png)

- windows 下安装

redis-server --service-install redis.windows-service.conf --loglevel verbose
		
.\redis-server.exe --service-install  --service-name redis3 --port 10001 --logfile server_log1001.txt --dbfilename dump10001.rdb
		
.\redis-server.exe --service-install .\redis.windows-service10001.conf --service-name redis3
		
redis-server --service-start --service-name redisService1

redis-server --service-stop
