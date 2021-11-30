## K8S集成Prometheus与EFK 

### 背景

```
k8s上线后的事情

```

### 前置条件

```
K8S集群搭建完成，服务均正常运行，整个集群都是使用ubuntu16.04(手动升级到了最新的内核)
```
### 版本说明

```
k8s-1.13.2
```

### 一、Prometheus 

#### 1.搭建nfs服务动态提供持久化存储 

```
1.安装nfs 
sudo apt-get install -y nfs-kernel-server
sudo apt-get install -y nfs-common 
sudo vi /etc/exports 
/data/opv *(rw,sync,no_root_squash,no_subtree_check)
注意将*换成自己的ip段，纯内网的话也可以用*，代替任意
sudo /etc/init.d/rpcbind restart 
sudo /etc/init.d/nfs-kernel-server restart 
sudo systemctl enable rpcbind nfs-kernel-server 

客户端挂载使用
sudo apt-get install -y nfs-common
mount -t nfs ku13-1:/data/opv  /data/opv -o proto=tcp -o nolock
为了方便使用将上面的mount命令直接放到.bashrc里面 
2.创建namesapce
kubectl create -f nfs/monitoring-namepsace.yaml 
3.为nfs创建rbac 
kubectl create -f nfs/rbac.yaml 
4.创建deployment,将文件里的nfs地址换成自己的
kubectl create -f nfs/nfs-deployment.yaml 
5.创建storageclass
kubectl create -f nfs/storageClass.yaml 
```
#### 新加了ceph rbd 持久化存储

```
具体部署使用方式参考rook-ceph目录下的README.md
```

#### 2.安装Prometheus 

```
1.创建权限
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
6.修改core-configmap里的etcd地址
```

#### 3.安装Grafana

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
