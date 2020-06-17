#### NFS 存储

- NFS 服务

```
# *on nfs-server*
# 启动服务
$ systemctl start nfs
$ systemctl status nfs

# 创建目录
$ mkdir -p /workspace/data/nfs0

# 修改配置
$ vi /etc/exports
/workspace/data/nfs0 *(rw,sync,insecure,no_subtree_check,no_root_squash)

# 生效
$ exportfs -r

# 重启rpcbind、nfs服务
$ systemctl restart rpcbind && systemctl restart nfs
# Error处理：
$ systemctl status rpcbind.socket
$ netstat -ntlp | grep 111

# 查看生效
$ showmount -e localhost
Export list for localhost:
/workspace/data/nfs0 *
```

```
# *on any client*
# 查看生效
$ showmount -e 192.168.10.101
Export list for 192.168.10.101:
/workspace/data/nfs0 *
```

- NFS 部署 on kubernetes

> Reference: https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client

```
# 1. rbac
# 增加 namespace: nfs，并编辑相关配置
$ vi nfs-rbac.yaml
$ kubectl apply -f nfs-rbac.yaml
```

```
# 2. deployment
$ vi nfs-deployment.yaml
$ kubectl apply -f nfs-deployment.yaml
```

```
# 3. storage-class & pvc
# storageclass
$ vi nfs-storage-class.yaml
$ kubectl apply -f nfs-storage-class.yaml

# pvc
$ vi nfs-pvc.yaml
$ kubectl apply -f nfs-pvc-test.yaml
```

---



#### glusterfs-kubernetes 存储

- 环境依赖

```
# 增加裸盘 /dev/sdb
$ fdisk -l
```

```
# 环境
modprobe  dm_snapshot
modprobe   dm_mirror
modprobe   dm_thin_pool

# 依赖
yum install centos-release-gluster
yum install -y glusterfs glusterfs-server glusterfs-fuse
```

- glusterfs 部署

```
# 打上 glusterfs 标签
$ kubectl label nodes --all storagenode=glusterfs
$ kubectl get nodes --show-labels
# 删除 label (命令行指定LabelName与减号相连)
$ kubectl label node centos-master-02 LabelName- 
 
# 部署 glusterfs 
$ kubectl apply -f glusterfs-daemonset.yaml
```

- heketi 部署及配置

```
# 建立 ServiceAccount 并赋予权限 
$ kubectl apply -f heketi-security.yaml

# 部署 heketi (利用 nodeSeletor 部署在 master-02 上)
$ kubectl apply -f heketi-deploy.yaml
```

```
# 进入 heketi 容器
$ docker exec -it ${heketi_container_id} bash

# ---- 以下是在容器内部 ---- #
$ export HEKETI_CLI_SERVER=http://localhost:8080

$ vi topology.json # 按照官方sample 修改

# 部署
$ heketi-cli -s $HEKETI_CLI_SERVER --user admin --secret 'My Secret' topology load --json=topology.json

# 查看
$ heketi-cli --user admin --secret 'My Secret' topology info

# 可查看/修改密码
$ vi /etc/heketi/heketi.json

$ heketi-cli --user admin --secret 'My Secret' cluster list
$ heketi-cli --user admin --secret 'My Secret' cluster delete ${cluster_id}
```

```
# 进入 gluster 容器
$ docker exec -it ${gluster_container_id} bash

# --- 容器内部 --- #
# 验证 gluster connected
$ gluster peer status
$ gluster volume info
```

- abort & reset 

```
# abort for gk-deploy 部署 
$ ./gk-deploy --abort -y -n glusterfs -g --user-key=userkey --admin-key=adminkey
$ kubectl delete ns glusterfs

# 下面命令是每个glusterfs集群需做的
rm -rf /etc/glusterfs/
rm -rf /var/log/glusterfs/
rm -rf /var/lib/glusterd/
rm -rf /var/lib/heketi/

# 删除 heketi 数据库(on master-02)
rm -rf /workspace/data/heketi-data

# 删除 Device Mapper
dmsetup status
dmsetup remove_all

# 格式化 /dev/sdb
dd if=/dev/zero of=/dev/sdb bs=512k count=1 
```

- StorageClass & PVC

```
# Storageclass 部署
$  kubectl -n glusterfs  describe svc/heketi | grep "Endpoints:" | awk '{print $2}'
$ vi glusterfs-storageclass.yaml # 修改 heketi_service 访问地址
$ kubectl apply -f glusterfs-storage-class.yaml

# PVC 部署
$ kubectl apply -f glusterfs-pvc-test.yaml
```

```
kubectl get all --all-namespaces -o wide | grep heketi
```