# 有需要就部署，没需要就不用管了

## 例子

### 部署mysql

```
1.创建
kubectl  create -f exportor/mysql 
2.修改prometheus的prometheus-core-configmap.yaml 
 - job_name: 'mysql-exporter' 
        static_configs:
        - targets: ['mysql-exporter:9104']

```
