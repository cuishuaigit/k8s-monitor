## K8S集成Prometheus与EFK 

### 背景

```
最近在做K8S的私有云项目建设，集群搭建功能基本完善，但是监控系统一直空缺，主要包括资源和日志两部分。看到部分私有云厂商使用了Prometheus来做资源的监控，包括网络、CPU、Memory等资源，所以私下也搭建了Prometheus集成K8S。
```

### 前置条件

```
K8S集群搭建完成，服务均正常运行
```

### 一、Prometheus 

#### 1.安装Prometheus 

```
1.创建命名空间monitoring
kubectl create -f namespace.yaml
2.创建权限
kubectl create -f rbac.yaml
2.创建 node-exporter
kubectl create -f prometheus-node-exporter-daemonset.yaml
kubectl create -f prometheus-node-exporter-service.yaml
3.创建 kube-state-metrics
kubectl create -f kube-state-metrics-deployment.yaml
kubectl create -f kube-state-metrics-service.yaml
4.创建 node-directory-size-metrics
kubectl create -f node-directory-size-metrics-daemonset.yaml
5.创建 prometheus
kubectl create -f prometheus-pvc.yaml
kubectl create -f prometheus-core-configmap.yaml
kubectl create -f prometheus-core-deployment.yaml
kubectl create -f prometheus-core-service.yaml
kubectl create -f prometheus-rules-configmap.yaml
```

#### 2.安装Grafana

```
1.安装grafana service
kubectl create -f grafana-svc.yaml
2.创建configmap
kubectl create -f grafana-configmap.yaml
3.创建pvc
kubectl create -f grafana-pvc.yaml
4.创建gragana deployment
kubectl create -f grafana-deployment.yaml
5.创建dashboard configmap
kubectl create configmap "grafana-import-dashboards" --from-file=dashboards/ --namespace=monitoring
6.创建job，导入dashboard等数据
 kubectl create -f grafana-job.yaml
```



### 二、EFK（Elasticsearch + Fluentd + Kibana）

```
EFK的安装方式参考 https://github.com/gjmzj/kubeasz/tree/master/manifests/efk，唯一需要注意的是Docker启动方式中，log-driver需要改成json-file的格式。
另外，安装Fluentd的时候，需要设置label，
kubectl label nodes XXXX beta.kubernetes.io/fluentd-ds-ready=true
```



### 相关文档：

https://www.cnblogs.com/163yun/p/7716253.html

https://www.cnblogs.com/sfnz/p/6566951.html

https://blog.csdn.net/wenwst/article/details/76624019

https://github.com/prometheus

https://prometheus.io/docs/

https://grafana.com

https://grafana.com/dashboards

https://github.com/gjmzj/kubeasz

https://github.com/giantswarm/kubernetes-prometheus

http://blog.51cto.com/xujpxm/2055970

http://yunlzheng.github.io/2018/03/12/prometheus-alertmanager-ha/