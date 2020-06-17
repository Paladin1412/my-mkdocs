- 安装

```
# 创建命名空间
$ kubectl create namespace istio-system

# 初始化，安装所有 CRDs
$ helm template helm/istio-init --name my-istio-init --namespace istio-system > istio-init.yaml
$ kubectl apply -f istio-init.yaml

# 安装组件
$ helm template helm/istio --name my-istio --namespace istio-system > istio.yaml
$ kubectl apply -f istio.yaml
```

```
kubectl -n istio-system get all -o wide
```

- Debug

```
# CPU/MEMORY 不足

# 查看 istio-pilot
$ kubectl -n istio-system get pod/istio-pilot-847c86787b-cwb84 -o yaml  | grep -E "cpu|memory|limits|requests"
# 查看 istio-telemetry
$ kubectl -n istio-system get  pod/istio-telemetry-7df546557f-7t59g -o yaml |  grep -E "cpu|memory|limits|requests"

# 手动调小
$ kubectl -n istio-system edit deployment.apps/istio-pilot
$ kubectl -n istio-system edit deployment.apps/istio-telemetry 
```

- Demo

```
# default 命名空间打上标签 istio-injection=enabled (默认自动注入 Sidecar)
$ kubectl label namespace default istio-injection=enabled
$ kubectl get namespaces --show-labels
```

```
# 部署应用
$ kubectl apply -f bookinfo.yaml
```

> Error: istio自动注入失败
>
> $ kubectl describe replicaset.apps/details-v1-6657b8bdf  # 发现错误
>
> 解决：安装 metrics-server

```
# 定义 Ingress 网关：
$ kubectl apply -f bookinfo-gateway.yaml
$ kubectl get gateway
```

```
# 访问验证
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')

$ env | grep INGRESS
$ curl -s http://${INGRESS_HOST}:${INGRESS_PORT}/productpage | grep -o "<title>.*</title>"
```

- 监控

```
# prometheus
$ vi helm/istio/charts/prometheus/values.yaml
# enable ingress，修改hosts，重新生成模板

# grafana
$ vi helm/istio/values.yaml
$ vi helm/istio/charts/grafana/values.yaml
```

- 网格可视化（Kiali）

```
# 创建 secret

$ KIALI_USERNAME=$(read -p 'Kiali Username: ' uval && echo -n $uval | base64)
$ KIALI_PASSPHRASE=$(read -sp 'Kiali Passphrase: ' pval && echo -n $pval | base64)
$ NAMESPACE=istio-system

$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: $NAMESPACE
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE
EOF
```

> admin/kiali12345

```
# kiali
$ vi helm/istio/values.yaml # enale kiali
$ vi helm/istio/charts/kiali/values.yaml # enale ingress

# 重新生成模板, apply 一下
```



- Tracing

```
# tracing
$ vi helm/istio/values.yaml # enable tracing
$ vi helm/istio/charts/tracing/values.yaml # enable ingress

# 安装组件
$ helm template helm/istio --name my-istio --namespace istio-system > istio.yaml
$ kubectl apply -f istio.yaml
```

