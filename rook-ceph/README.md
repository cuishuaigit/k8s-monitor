# This repo is  about that deploy  ceph-rbd as persistent storage

```
  rook version: 1.0.2
  ceph version: v14.2.1-20190430
  kubernetes version: 1.15.0
  system: ubuntu 16.04 4.17.0-041700-generic #201806041953 SMP Mon Jun 4 19:55:25 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

## Deploy ceph cluster with rook

```bash
git clone https://github.com/rook/rook.git
cd rook/cluster/examples/kubernetes/ceph
kubectl apply -f common.yaml 
kubectl apply -f operator.yaml 
```
when all pods is running, Next need to deploy cluster.yaml, before that we need update some config.

```
dataDirHostPath: shuold mount  a single disk
useAllNodes: set it to false,we need set nodeselector 
useAllDevices: set it to false, we need mount a single disk
nodes: set  which node will use to deploy ceph 
```

```bash
kubectl apply -f cluster.yaml 
```
there was a bug in rook, when you use dashboard, ceph now not suport well weith iscsi. so you need modify default administrator role 
set with no iscsi in rook-ceph-operator pod:

```bash
kubectl exec -it rook-ceph-operator-56b7cdb77b-v6r68 -n rook-ceph -- bash
```


exec the follow command in the pod:

```bash
   ceph dashboard ac-role-create admin-no-iscsi

   for scope in dashboard-settings log rgw prometheus grafana nfs-ganesha manager hosts rbd-image config-opt rbd-mirroring cephfs user osd pool monitor; do
       ceph dashboard ac-role-add-scope-perms admin-no-iscsi ${scope} create delete read update;
   done

   ceph dashboard ac-user-set-roles admin admin-no-iscsi
```

restart mgr pod:

```bash
 kubectl delete pod rook-ceph-mgr-a-dd68b7dff-4wnzm -n rook-ceph
```
the dashboard default  user is admin, the password you can get it in secret:

```bash
paw=`kubectl get secret rook-ceph-dashboard-password -n rook-ceph -o yaml | grep password: | awk -F ":" '{print $2}`
echo $paw | base64 -d
```
you need create a new service to access the dashboard in external:

```bash
kubectl apply -f dashboard-external-https.yaml
```

create a storageclass:

```bash
 kubectl create -f storageclass.yaml 
```
test ceph cluster:

```bash
kubeclt create -f test-ceph-ebd.yaml 
kubectl exec -it nginx-pod1 -- echo "Hello ceph rbd" > /usr/share/nginx/html/index.html
curl pod-ip 
Hello ceph rbd
```
## Dynamic expand ceph cluster

```bash
kubectl edit cephcluster rook-ceph -n rook-ceph
```

```
nodes:
   - config: null
      name: ceph-node-4
      resources: {}
    - config: null
      name: ceph-node-5
      resources: {}
    - config: null
      name: ceph-node-6
      resources: {}
```
