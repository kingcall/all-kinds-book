## 集群基本信息
- curl -X GET http://localhost:8081/v1/overview | jq .
```
{
  "taskmanagers": 3,
  "slots-total": 6,
  "slots-available": 6,
  "jobs-running": 0,
  "jobs-finished": 0,
  "jobs-cancelled": 0,
  "jobs-failed": 0,
  "flink-version": "1.9.0",
  "flink-commit": "9c32ed9"
}
```
## jobmanager config
- curl -X GET http://localhost:8081/v1/jobmanager/config | jq .

## 获取所有的taskmanager
- curl -X GET http://localhost:8081/v1/taskmanagers | jq

## 获取所有的metric
- curl -X GET http://localhost:8081/v1/jobmanager/metrics | jq