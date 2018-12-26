### Redis-exporter安装步骤

```
1.修改Redis-exporter地址：
按照要求修改yaml文件，修改redis-exporter-deployment.yaml以下内容，
args: ["-redis.addr", "svc-redis-cluster-np.default:6379"] 
将svc-redis-cluster-np.default:6379修改成自己集群内部的Redis访问负载地址，相应参数参考redis_exporter启动命令。
```



