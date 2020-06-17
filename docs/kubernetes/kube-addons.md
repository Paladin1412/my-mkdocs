### Helm

```
# 安装
$ curl -L https://git.io/get_helm.sh | bash
```

```
# 创建服务账号和角色绑定
$ cat <<EOF > ./rbac-config.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
EOF

$ kubectl create -f rbac-config.yaml
```

```
# helm Chart 仓库改成阿里云 Chart 镜像仓库
$ helm repo remove stable
$ helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

# helm init 初始化
$ helm init --service-account tiller -i registry.aliyuncs.com/google_containers/tiller:v2.14.3 # 安装
$ helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.19.1 # 升级版本
```

```
# 查看安装成功
$ helm  version
```

- 其他命令

```
 $ helm repo add stable http://mirror.azure.cn/kubernetes/charts
 $ helm repo update
 $ helm repo list
```

----



####  Ingress-nginx

```
https://raw.githubusercontent.com/helm/charts/master/stable/nginx-ingress/values.yaml

$ vi ingress-nginx-values.yaml

修改：
# kind: Deployment => kind: DaemonSet
# hostNetwork: false => hostNetwork: true
# dnsPolicy: ClusterFirst => dnsPolicy: ClusterFirstWithHostNet
# type: LoadBalancer => type: ClusterIP
```

```
# 安装
$ helm install stable/nginx-ingress --name nginx-ingress -f ingress-nginx-values.yaml --namespace ingress-nginx

# 拉取镜像失败 error： 手动拉取

# 删除
$ helm ls --all nginx-ingress
$ helm del --purge nginx-ingress
```

````
# 验证部署成功
$ curl http://192.168.10.100:80 (集群内任意ip)
default backend - 404
````

---



#### Kubernetes dashboard

```
# 部署
$ kubectl apply -f kube-dashboard.yaml
$ kubectl apply -f kube-dashboard-auth.yaml

# 验证部署成功(on master-01)
$ kubectl proxy
$ curl http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

```
# 获取Bearer Token
$ kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

-  Ingress 访问 kubernetes dashboard（ HTTPS 访问）

```
# 生成自签名证书
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout kube-dashboard.key -out kube-dashboard.crt -subj "/CN=dashboard.kube.com/O=dashboard.kube.com"
```

```
# 使用自签名证书创建 k8s Secret 资源，供 Ingress 引用
$ kubectl create secret tls kube-dasboard-ssl --key kube-dashboard.key --cert kube-dashboard.crt -n kube-system
```

```
# 配置
$ vi kube-dashboard-ingress.yml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-kube-dashboard
  namespace: kubernetes-dashboard
  annotations:
    # use the shared ingress-nginx
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  tls:
  - hosts:
    - dashboard.kube.com
    secretName: kube-dasboard-ssl
  rules:
  - host: dashboard.kube.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kubernetes-dashboard
          servicePort: 443

# 部署
$ kubectl kube-dashboard-ingress.yml
```

```
# 查看 ingress
$ kubectl get ingress --all-namespaces -o wide
```

---



####  metrics-server

```
https://raw.githubusercontent.com/helm/charts/master/stable/metrics-server/values.yaml 

$ vi metrics-values.yaml

修改：
args:
- --logtostderr
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP
```

```
# 安装
$ helm install stable/metrics-server \
-n metrics-server \
--namespace kube-system \
-f metrics-values.yaml

# 卸载
# helm del --purge metrics-server 
```

```
$ kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes"
$ kubectl top node
```

- Debug

```
# 1. metrics-server-deployment.yaml 添加参数
- --kubelet-insecure-tls

# 2. kube-apiserver 添加参数，配置 Aggregation
- --enable-aggregator-routing=true

# 3. 添加hosts
kubectl edit configmap coredns -n kube-system
```

