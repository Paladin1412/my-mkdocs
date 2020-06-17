### 环境准备

- 安装依赖

```
# 更新yum
$ yum update -y
# 安装依赖包
$ yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp bind-utils lvm2
```

```
# 安装 docker
$ yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
 
 $ yum install -y docker-ce
```

```
# 设置docker启动参数（可选）
$ cat <<EOF > /etc/docker/daemon.json
{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

# 启动docker服务
$ service docker restart
```
```
# 配置yum源（科学上网的同学可以把"mirrors.aliyun.com"替换为"packages.cloud.google.com"）
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 找到要安装的版本号
$ yum list kubeadm --showduplicates | sort -r
# 安装工具
$ yum install -y kubeadm kubelet kubectl --disableexcludes=kubernetes
```

- 关闭防火墙、swap、selinux、dnsmasq、重置iptables

```
# 关闭防火墙
$ systemctl stop firewalld && systemctl disable firewalld

# 重置iptables
$ iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT
# 查看 
$ iptables -L -n
$ iptables-save

# 关闭swap
$ swapoff -a
$ sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab

# 关闭selinux
$ setenforce 0
$ getenforce # 查看状态
$ vi /etc/sysconfig/selinux # 改为disabled

# 关闭dnsmasq(否则可能导致docker容器无法解析域名)
$ service dnsmasq stop && systemctl disable dnsmasq
```

- 开启ipvs

```
# 开启ipvs模式(需要注意的是要添加ip_vs相关模块)

cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
modprobe br_netfilter
EOF

# 生效
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

```
# 制作配置文件
$ cat > /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
net.ipv4.tcp_tw_recycle=1 # 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_keepalive_time=600 # 超过这个时间没有数据传输，就开始发送存活探测包
net.ipv4.tcp_keepalive_intvl=15 # keepalive探测包的发送间隔
net.ipv4.tcp_keepalive_probes=3 # 如果对方不予应答，探测包的发送次数
vm.swappiness=0 # 禁止使用 swap 空间，只有当系统 OOM 时才允许使用它
vm.overcommit_memory=1 # 不检查物理内存是否够用
vm.panic_on_oom=0 # 开启 OOM
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=1048576
fs.file-max=52706963
fs.nr_open=52706963
net.ipv6.conf.all.disable_ipv6=1
net.netfilter.nf_conntrack_max=2310720
EOF

# 生效文件
$ sysctl -p /etc/sysctl.d/kubernetes.conf
```
- 其他

```
# 调整系统 TimeZone
$ timedatectl set-timezone Asia/Shanghai
# 将当前的 UTC 时间写入硬件时钟
$ timedatectl set-local-rtc 0
# 重启依赖于系统时间的服务
$ systemctl restart rsyslog && systemctl restart crond

# 关闭无关的服务
$ systemctl stop postfix && systemctl disable postfix
```

---



### 高可用 HA

- keepalived

```
# vip: 192.168.10.100
# master-01:  192.168.10.101 (主)
# master-02:  192.168.10.102 （备）
```

```
# 安装
$ yum install -y keepalived

# 配置（查看文档）
$ vi /etc/keepalived/keepalived.conf

# 启动
$ systemctl enable keepalived && systemctl start keepalived

# 检查
$ ip a
$ ping 192.168.10.100 # ping 不通 可能与防火墙有关(firewalld | selinux)
```

---



### 集群配置

- master-01

```
# 查看默认配置
$ kubeadm config print init-defaults
```

```
$ cd /workspace/kubernetes/configs/

# 生成配置文件
export APISERVER_NAME=192.168.10.100
# export POD_SUBNET=10.100.0.1/16 # for calico
export POD_SUBNET=10.244.0.0/16 # for flannel

cat <<EOF > ./kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: "${APISERVER_NAME}"
  bindPort: 6443
nodeRegistration:
  taints:
  - effect: PreferNoSchedule
    key: node-role.kubernetes.io/master
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.17.2
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
controlPlaneEndpoint: "${APISERVER_NAME}:6443"
networking:
  serviceSubnet: "10.97.0.0/16"
  podSubnet: "${POD_SUBNET}"
  dnsDomain: "cluster.local"
apiServer:
  certSANs:
  - 192.168.10.100
  - 192.168.10.101
  - 192.168.10.102
  - 192.168.10.121
  - 192.168.10.122
etcd:
  external:
    endpoints:
    - https://192.168.10.101:2379
    - https://192.168.10.102:2379
    caFile: /etc/kubernetes/pki/etcd/ca.pem
    certFile: /etc/kubernetes/pki/etcd/etcd.pem
    keyFile: /etc/kubernetes/pki/etcd/etcd-key.pem
EOF
```

```
# 初始化集群
kubeadm init --config=kubeadm-config.yaml --upload-certs 
```

```
# api-server
$ echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
$ source ~/.bash_profile
```

- 安装flannel网络插件 (on master-01)

```
$ cd /workspace/kubernetes/configs/

# 下载文件（并修改 image 源）
$ curl -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml /workspace/kubernetes/configs/

# 安装
$ kubectl apply -f kube-flannel.yml
```

> for vagrant:  
>
> eth0 网卡： nat 转发访问公网（默认）；
>
> eth1 网卡：主机真正的 IP，cluster 内部通信。
>
> ```
> vi /workspace/kubernetes/configs/kube-flannel.yml
> 
> ...
> - --kube-subnet-mgr
> - --iface=eth1 // 添加此行
> ```

- 修改kube-proxy 的 mode 为 ipvs  (on master-01)

```
$ kubectl edit cm kube-proxy -n kube-system
...
kind: KubeProxyConfiguration
mode: "ipvs" // 修改此处
```

```
kubectl get pod -n kube-system | grep kube-proxy |awk '{system("kubectl delete pod "$1" -n kube-system")}'
```

-  master-02

```
 $  kubeadm join 192.168.10.100:6443 --token cwn7o6.65gr67a36hdn992i \
    --discovery-token-ca-cert-hash sha256:c021540697fc731ffec34d0a18681f89562ca29a808f1ff9ad941a364b400281 \
    --control-plane --certificate-key 85b550cf7e1c6bfa3cdb626615722f6efe39cbe24e9f1f37da985685a8a3c643
```

> etcd health error

>  kubectl exec -it etcd-centos-master-01 sh -n kube-system
>
> ETCDCTL_API=3 alias etcdctl='etcdctl --endpoints=https://192.168.10.101:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key'
>ETCDCTL_API=3 etcdctl member list

- worker

```
# 查看加入 worker 
kubeadm token create --print-join-command

# 加入集群
$ kubeadm join 192.168.10.100:6443 --token  7zwmn0.k1m7i5docr5rkv49 \
    --discovery-token-ca-cert-hash sha256:c021540697fc731ffec34d0a18681f89562ca29a808f1ff9ad941a364b400281 
```

### 常用命令

```
# 重置
kubeadm reset

rm -rf /var/lib/etcd/*
rm -rf /etc/cni/
rm -rf /var/lib/cni/
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
ifconfig kube-ipvs0 down 
ip link delete kube-ipvs0

# etcd 证书
mkdir /etc/kubernetes/pki/etcd/
cp /workspace/kubernetes/ssl/*.pem /etc/kubernetes/pki/etcd/

# 查看
kubectl get nodes -o wide
kubectl get ep kubernetes
kubectl get all --all-namespaces -o wide

# 内容
kubectl -n kube-system describe  pod/coredns-667f964f9b-qngl5

# 日志
kubectl -n kube-system logs  pod/coredns-667f964f9b-qngl5
```

### TO-DO-LIST

- centos 7.x 内核版本 3.10.x 存在bugs, 会导致 Docker，Kubernetes 运行不稳定。 需要升级到 4.44 版本。

```
# 查看内核版本
$ uname -r
```

- Kubeadm 证书默认有效期一年，需要延长。

- 高可用架构方案升级。

> Keepalived + HAProxy

