[TOC]

## 前言

一般的hive客户端都没有详细的执行日志，要了解执行情况得去yarn看，就比较麻烦，本次考虑的是在hive客户端获取到全部的执行日志，来源主要参考hadoop-yarn-client的内部实现。

使用引擎为tez，在异步执行hql时，获取到的日志中包括applicationId，考虑通过这个去获取到全部的执行日志，包括进度和map/reduce信息。

## 实现

### jdbc部分日志

获取jdbc的执行日志调研过程中主要使用的是pyhive，（impyla在get_log的时候好像会出点问题）

以下为获取hive在客户端执行日志的部分代码，执行使用了异步的方式

```
from pyhive import hive
from TCLIService.ttypes import TOperationState

# 连接
conn = hive.connect(host=host, port=port, username=user,  database='default')
cursor = conn.cursor()

# 异步执行hql
cursor.execute('''select count(1) from table''', async_=True)

# 获取执行日志
# 每次poll拿到状态，如果还在执行中就fetch_logs并打印
# poll比较慢，建议测试时可以选择多join几张表
status = cursor.poll().operationState
application_id = None
while status in (TOperationState.INITIALIZED_STATE, TOperationState.RUNNING_STATE):
    logs = cursor.fetch_logs()
    for message in logs:
        print(message)
        match_res = re.findall(r'App id (.*?)\)', message)
        if len(match_res) > 0:
            application_id = match_res[0]
            break
    status = cursor.poll().operationState

cursor.fetchall()
cursor.close()
conn.close()
```

打印的日志大致如下，最后一行有applicationId，可以通过对每行进行正则匹配获取。

```
INFO  : Tez session hasn't been created yet. Opening session
DEBUG : Adding local resource: scheme: "hdfs" host: "NameNodeHACluster" port: -1 file: "/tmp/hive/hive/_tez_session_dir/a3b39087-cf39-4850-a8a0-2e31857a9a64/hive-contrib.jar"
DEBUG : DagInfo: {"context":"Hive","description":"select count(1) from table"}
DEBUG : Setting Tez DAG access for queryId=hive_20200409162528_98663859-dda3-4f6f-a56b-618dc92b5a0c with viewAclString=*, modifyStr=souche,hive
INFO  : Setting tez.task.scale.memory.reserve-fraction to 0.30000001192092896
INFO  : Status: Running (Executing on YARN cluster with App id application_1584028893195_1234)
```

### 进度和mapreduce信息

之后考虑进度和mapreduce信息日志，这个在hive客户端执行的时候是有一张表格展示的。

先看下面poll的源码解释，返回的是TGetOperationStatusResp，在追踪到这个,除了上面用到的拿到目前的执行状态operationState以外，还有一个叫progressUpdateResponse的，目测是想要的进度信息。

```
def poll(self, get_progress_update=True):
    """Poll for and return the raw status data provided by the Hive Thrift REST API.
    :returns: ``ttypes.TGetOperationStatusResp``
    :raises: ``ProgrammingError`` when no query has been started
    .. note::
        This is not a part of DB-API.
    """
class TGetOperationStatusResp(object):
    """
    Attributes:
     - status
     - operationState
     - sqlState
     - errorCode
     - errorMessage
     - taskStatus
     - operationStarted
     - operationCompleted
     - hasResultSet
     - progressUpdateResponse
    """
```

然后修改上面poll部分的代码，得到进度和mapreduce信息，
tabulate为画表格库

```
poll = cursor.poll()
status = poll.operationState
application_id = None
while status in (TOperationState.INITIALIZED_STATE, TOperationState.RUNNING_STATE):
    arr = []
    # 获取poll中的progressUpdateResponse
    # headerNames里为头信息，rows里为每行的数据
    arr.append(poll.progressUpdateResponse.headerNames)
    arr.extend(poll.progressUpdateResponse.rows)
    print(tabulate(arr, tablefmt='grid'))
    print("progress: {}%".format(round(poll.progressUpdateResponse.progressedPercentage * 100, 2)))
    poll = cursor.poll()
    status = poll.operationState
```

效果如下
![pyhiveprogress](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/pyhiveprogress.png)

### 全部conrainer日志

继续考虑获取从yarn获取全部日志，一般的可以在集群中用以下命令获取全部日志,containerId可以从yarn管理页面获取到。

> yarn logs -applicationId {applicationId}

> yarn logs -containerId {containerId}

然后查看hadoop-yarn-client中对上面命令实现的部分，从中找出具体的日志接口。包和文件路径为

> org.apache.hadoop.yarn.client.cli.LogsCLI

函数追踪的路径如下

> runCommand -> fetchAMContainerLogs -> printAMContainerLogs -> getAMContainerInfoForRMWebService -> getAMContainerInfoFromRM

然后可以看到如下部分代码，这边包装了一个get请求，因此也按它的地址调用一下

> [http://host](http://host/):port/ws/v1/cluster/apps/{applicationId}/appattempts

```
Builder builder = webServiceClient.resource(webAppAddress)
          .path("ws").path("v1").path("cluster")
          .path("apps").path(appId).path("appattempts")
          .accept(MediaType.APPLICATION_JSON);
response = builder.get(ClientResponse.class);
JSONObject json = response.getEntity(JSONObject.class)
    .getJSONObject("appAttempts");
JSONArray requests = json.getJSONArray("appAttempt");
```

得到数据结构大致如下，logsLink直接访问就是log的html的地址
这边记录下containerId和nodeHttpAddress

```
{
    "appAttempts": {
        "appAttempt": [
            {
                "id": 1,
                "startTime": 1585100799481,
                "finishedTime": 1585100821657,
                "containerId": "container_e45_000001",
                "nodeHttpAddress": "host:port",
                "nodeId": "host:port",
                "logsLink": "http://host:port/node/containerlogs/conta801/hive",
                "blacklistedNodes": "",
                "appAttemptId": "appattempt_15840200001"
            }
        ]
    }
}
```

然后追踪另一条路，又可以发现它获取日志的地方

> runCommand -> fetchContainerLogs -> getMatchedOptionForRunningApp -> getMatchedContainerLogFiles -> getContainerLogFiles

```
WebResource webResource = webServiceClient
          .resource(WebAppUtils.getHttpSchemePrefix(conf) + nodeHttpAddress);
ClientResponse response =
    webResource.path("ws").path("v1").path("node").path("containers")
        .path(containerIdStr).path("logs")
        .accept(MediaType.APPLICATION_JSON)
        .get(ClientResponse.class);
```

这边的地址拼接如下

> http://{nodeHttpAddress}/ws/v1/node/containers/{containerId}/logs

访问后可以获取到如下信息
containerLogInfo里面每个都是日志文件。

```
[
    {
        "containerId": "container_e455_01",
        "nodeId": "hadoop-4355",
        "containerLogInfo": [
            {
                "fileName": "dag_1584028893195_0587_1.dot",
                "fileSize": "1631",
                "lastModifiedTime": "Wed Mar 25 09:47:03 +0800 2020"
            },
            {
                "fileName": "directory.info",
                "fileSize": "18349",
                "lastModifiedTime": "Wed Mar 25 09:47:03 +0800 2020"
            },
            ......
        ],
        "logAggregationType": "AGGREGATED"
    }
]
```

可以在前面的地址后加上其中的文件名获取到具体每个log文件的内容，地址如下，get调用即可

> http://{nodeHttpAddress}/ws/v1/node/containers/{containerId}/logs/{fileName}

至此已获取到全部日志。