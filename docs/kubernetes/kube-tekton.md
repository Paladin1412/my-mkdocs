## Install

- 安装 Tekton Pipelines Core

```
$ kubectl apply -f tekton-core-release.yaml
```

```
# Set up the CLI
$ tar xvzf tkn_0.9.0_Linux_x86_64.tar.gz -C /usr/local/bin/ tkn
```

- 安装  Tekton Triggers

```
$ kubectl apply -f tekton-triggers-release.yaml
$ kubectl get all -n tekton-pipelines
$ kubectl api-resources | grep tekton
```

- 安装  Tekton Dashboard

```
# install dashboard
$ kubectl apply -f tekton-dashboard-release.yaml
# 改成 nodePort 模式
$ curl http://10.62.250.33:25955
```

## Tekton CRDs

```
$ kubectl api-resources | grep tekton
```

- Tekton Trigger 
  - `TriggerTemplate`: 创建资源的模板(比如用来创建 PipelineResource 和 PipelineRun)；
  - `TriggerBinding`: 校验事件，提取负载字段；
  - `EventListener`: 连接`TriggerBinding`和`TriggerTemplate`到可寻址的端点(事件接收器)。使用从各个` TriggerBinding`中提取的参数来创建`TriggerTemplate`中指定的 resources. 同样通过` interceptor`字段来指定外部服务来对事件负载进行预处理；
  - `ClusterTriggerBinding`: cluster级别的`TriggerBinding`。

## Examples

- Task Demo

```
# task demo
$ kubectl apply -f task-demo.yaml

# run task
$ tkn task start echo # either
$ kubectl apply -f taskrun-demo.yaml # or

# see the logs
$ kubectl get pod
$ tkn taskrun describe taskrun-demo
$ tkn taskrun logs -f taskrun-demo
```

- Pipeline Demo

```
# Kaniko Config
$ vi config.json

{"auths":{"docker.io":{"username":"siverbulllet","password":"cool19900926"},"registry.cn-hangzhou.aliyuncs.com":{"username":"leonardochen10@163.com","password":"cool19900926"}}}

$ kubectl create configmap docker-config --from-file=config.json
```

```
# 常用命令
$ tkn pipelinerun logs --last -f
```

```
$ kubectl proxy --port=8001
# curl localhost:8001 
# nginx 代理一下 8001 => 8009： http://10.62.252.160:8009/

$ curl http://localhost:8001/api/v1/namespaces/[namespace-name]/services/[service-name]/proxy
```



```
---
apiVersion: triggers.tekton.dev/v1alpha1
kind: EventListener
metadata:
  name: github-listener-interceptor
spec:
  serviceAccountName: tekton-triggers-github-sa
  triggers:
    - name: github-listener
      interceptors:
        - github:
            secretRef:
              secretName: github-secret
              secretKey: secretToken
            eventTypes:
              - pull_request
      bindings:
        - ref: github-binding
      template:
        name: github-template
```



```
URL="https://github.com/silverbulllet/myapp-demo" # The URL of your fork of CatApp
ROUTE_HOST="http://trigger-test.kube.com"
curl -v \
   -H 'X-GitHub-Event: pull_request' \
   -H 'Content-Type: application/json' \
   -d '{
     "repository": {"clone_url": "'"${URL}"'"},
     "pull_request": {"head": {"sha": "master"}}
   }' \
   ${ROUTE_HOST}
```

