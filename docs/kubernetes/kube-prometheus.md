#### Prometheus

> https://github.com/helm/charts/tree/master/stable/prometheus-operator

- 部署

```
# 安装
$ cd /workspace/kubernetes/addons/prometheus
$ helm install ./prometheus-operator/ --name my-prome --namespace monitoring
```

```
# 删除
$ helm delete --purge my-prome

# 删除crd资源
$ kubectl get crd | grep monitoring
kubectl delete crd prometheuses.monitoring.coreos.com
kubectl delete crd prometheusrules.monitoring.coreos.com
kubectl delete crd servicemonitors.monitoring.coreos.com
kubectl delete crd podmonitors.monitoring.coreos.com
kubectl delete crd alertmanagers.monitoring.coreos.com
kubectl delete crd thanosrulers.monitoring.coreos.com  
```

```
# 更新
helm upgrade my-prome ./prometheus-operator/ -f ./prometheus-operator/charts/prometheus-node-exporter/values.yaml
```

```
# 获取 grafana 登陆密码
$ kubectl -n monitoring get secret my-prome-grafana -o yaml

$ echo cHJvbS1vcGVyYXRvcg== | base64 -d
prom-operator

$ echo YWRtaW4= | base64 -d
admin
```

- 常用命令

```
# 查看镜像
$ find prometheus-operator/ | grep  values.yaml | xargs grep image:

# 查看
$ kubectl -n monitoring get all -o wide
```

- Debug

```
# 修改监听 target: etcd
# 1. 修改endpoints

# 2. http => https

# 3.添加证书
$ kubectl create secret generic etcd-certs -n monitoring --from-file=/etc/kubernetes/pki/etcd/ca.pem  --from-file=/etc/kubernetes/pki/etcd/etcd.pem  --from-file=/etc/kubernetes/pki/etcd/etcd-key.pem 

# 4. upgrade 配置
$ helm upgrade my-prome ./prometheus-operator/ -f ./prometheus-operator/values.yaml
```

```
# 修改监听 target: kube-proxy
$ netstat -ntlp | grep kube-proxy

$ kubectl -n kube-system edit cm kube-proxy
# ...
kind: KubeProxyConfiguration
# metricsBindAddress: 127.0.0.1:10249
metricsBindAddress: 0.0.0.0:10249
# ...

# 重启kubelet、docker服务，再netstat -ntlp | grep kube-proxy查看
```

