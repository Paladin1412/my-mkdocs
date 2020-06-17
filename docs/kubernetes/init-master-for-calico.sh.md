init-master.sh

```
#!/bin/bash

# 只在 master 节点执行

# 替换 x.x.x.x 为 master 节点实际 IP（请使用内网 IP）
# export 命令只在当前 shell 会话中有效，开启新的 shell 窗口后，如果要继续安装过程，请重新执行此处的 export 命令
export MASTER_IP=192.168.10.101
# 替换 apiserver.demo 为 您想要的 dnsName
# export APISERVER_NAME=apiserver.demo
export APISERVER_NAME=192.168.10.100
# Kubernetes 容器组所在的网段，该网段安装完成后，由 kubernetes 创建，事先并不存在于您的物理网络中
# export POD_SUBNET=10.100.0.1/16 # for calico
export POD_SUBNET=10.244.0.0/16 # for flannel
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts
curl -sSL https://kuboard.cn/install-script/v1.17.x/init_master.sh | sh -s 1.17.2

## --- ##

# 脚本出错时终止执行
set -e

if [ ${#POD_SUBNET} -eq 0 ] || [ ${#APISERVER_NAME} -eq 0 ]; then
  echo -e "\033[31;1m请确保您已经设置了环境变量 POD_SUBNET 和 APISERVER_NAME \033[0m"
  echo 当前POD_SUBNET=$POD_SUBNET
  echo 当前APISERVER_NAME=$APISERVER_NAME
  exit 1
fi


# 查看完整配置选项 https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2
rm -f ./kubeadm-config.yaml
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

# kubeadm init
# 根据您服务器网速的情况，您需要等候 3 - 10 分钟
kubeadm init --config=kubeadm-config.yaml --upload-certs

# 配置 kubectl
rm -rf /root/.kube/
mkdir /root/.kube/
cp -i /etc/kubernetes/admin.conf /root/.kube/config

# 安装 calico 网络插件
# 参考文档 https://docs.projectcalico.org/v3.10/getting-started/kubernetes/
echo "安装calico-3.10.2"
rm -f calico-3.10.2.yaml
wget https://kuboard.cn/install-script/calico/calico-3.10.2.yaml
sed -i "s#192\.168\.0\.0/16#${POD_SUBNET}#" calico-3.10.2.yaml
kubectl apply -f calico-3.10.2.yaml
```