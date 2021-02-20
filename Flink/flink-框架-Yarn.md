## on-yarn 的优势
- 基于 YARN 调度系统，能够⾃动处理各个⻆⾊的 failover（Job Manager 进程异常退出，Yarn ResourceManager 会重新调度 Job Manager 到其他机器；如果 Task Manager 进程异常退出， Job Manager 收到消息后并重新向 Yarn ResourceManager 申请资源，重新启动 Task Manager）
## on-yarn 的配置
### 客户端
- 如果我们打算把flink任务提交到yarn上，那么只需要一台flink客户机即可，但是需要有如下的配置
> 设置 HADOOP_CONF_DIR环境变量 export HADOOP_CONF_DIR=/usr/local/service/hadoop/etc/hadooop/ 或者在配置文件中添加
> env.yarn.conf.dir: /usr/local/service/hadoop/etc/hadoop
```
1. 会查看YARN_CONF_DIR，HADOOP_CONF_DIR或者HADOOP_CONF_PATH是否设置，按照顺序检查的。然后，假如配置了就会从该文件夹下读取配置
2. 如果上面环境变量都没有配置的话，会使用HADOOP_HOME环境变量。对于hadoop2的话会查找的配置路径是 $HADOOP_HOME/etc/hadoop;对于hadoop1会查找的路径是$HADOOP_HOME/conf
```
### 设置提交用户名
```
IFS=' ' arr=($@)
for(( i=0;i<${#arr[@]};i++)) do
    if [ "-yD" = ${arr[i]} ]; then
        let i++
        IFS='=' arr2=(${arr[i]})
        if [ "name" = ${arr2[0]} ]; then
            export HADOOP_USER_NAME=${arr2[1]}
        fi
    fi
done
```
- -yD name=wnagzhxxx(指定任务用户)

## on-yarn 的流程
- 检查环节变量的配置
- 客户端会首先检查要请求的资源(containers和memory)是否可用。然后，将包含flink相关的jar包盒配置上传到hdfs
- 客户端会向resourcemanager申请一个yarncontainer用以启动ApplicationMaster(Jobmanager和AM运行于同一个container)
- AM开始申请启动FlinkTaskmanager的containers，这些container会从hdfs上下载jar文件和已修改的配置文件。