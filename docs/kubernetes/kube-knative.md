- 安装 Serving

```
# Install CRDs
$ kubectl apply --filename serving-crds.yaml

# Install core
$ kubectl apply --filename serving-core.yaml

# Install Knative Istio controlle
$ kubectl apply --filename net-istio-release.yaml
```

- 验证 Serving

```
# 部署 demo
$ kubectl apply -f hello-world.yaml

# 访问
export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

$ curl -H 'Host: helloworld-nodejs.default.example.com' http://192.168.10.121:31380
```

- Troubleshooting

> $ kubectl describe revision.serving.knative.dev/helloworld-go-2
>
> Get https://hub.chenleileicode.com:8080/v2/: http: server gave HTTP response to HTTPS client

```
$ kubectl -n knative-serving edit configmap config-deployment
...
registriesSkippingTagResolving: "ko.local,dev.local,hub.chenleileicode.com:8080"
```

- 安装 Eventing

```
# Install CRDs
$ kubectl apply --filename eventing-crds.yaml

# Install core
$ kubectl apply --filename eventing-core.yaml

# install Channel layer for messaging
$ kubectl apply -f in-memory-channel.yaml

# Install a Broker
$ kubectl apply --filename channel-broker.yaml
```

- 安装 monitoring（可视化插件）

```
# Install core
kubectl apply --filename monitoring-core.yaml

# Prometheus
$ kubectl apply -f monitoring-metrics-prometheus.yaml

# ELK
$ kubectl apply -f monitoring-logs-elasticsearch.yaml

#  Jaeger
$ kubectl apply -f monitoring-tracing-jaeger-in-mem.yaml

# Zipkin 
$ monitoring-tracing-zipkin-in-mem.yaml
```

- 安装 tekton

> 构建工具，独立于以上knative核心模块。

