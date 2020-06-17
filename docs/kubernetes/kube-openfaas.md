- 安装 OpenFaaS on K8s

```
$ git clone https://github.com/openfaas/faas-netes
```

```
# 命名空间
$ cd faas-netes && \
$ kubectl apply -f namespaces.yml
```

```
# Create a password for the gateway
$ kubectl -n openfaas create secret generic basic-auth \
--from-literal=basic-auth-user=admin \
--from-literal=basic-auth-password=admin
```

```
# deploy OpenFaaS
$ kubectl apply -f ./yaml
```

```
# ingress openfaas gateway
$ vi ./yaml/gateway-svc.yml
...
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-gateway
  namespace: openfaas
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: openfaas.kube.com
    http:
      paths:
      - path: /
        backend:
          serviceName: gateway
          servicePort: 8080
```

- 安装  faas-cli 工具

```
$ curl -sL https://cli.openfaas.com | sudo sh
```

- 运行 OpenFaaS Function

```
# 创建 hello-world function
$ faas-cli new --lang dockerfile hello-world --prefix=hub.chenleileicode.com

# 创建 hello-world-2 function
$ faas-cli new --lang node hello-world-2 --prefix=hub.chenleileicode.com:8080/serverless
```

```
# 登陆
$ faas-cli login -g openfaas.chenleileicode.com:8080 -u admin -p admin
```

```
# 部署
$ faas-cli up --yaml hello-world.yml
# 同上（分步执行）
$ faas-cli build -f ./hello-world.yml
$ faas-cli push -f ./hello-world.yml
$ faas-cli deploy -f ./hello-world.yml
$ faas-cli remove -f ./hello-world.yml
```

- Troubleshooting

> **Error**: rpc error: code = Unknown desc = Error response from daemon: pull access denied for hub.chenleileicode.com
>
> **Answer: **If you try to deploy using `faas-cli deploy` it will fail because the Kubernetes kubelet component will not have credentials to authorize the docker image pull request.

```
export DOCKER_USERNAME=xxx
export DOCKER_PASSWORD=xxx
export DOCKER_EMAIL=xxx
export DOCKER_SERVER=hub.chenleileicode.com
export DOCKER_SERVER2=hub.chenleileicode.com:8080

$ kubectl create secret docker-registry my-private-repo2 \
  --docker-username=$DOCKER_USERNAME \
  --docker-password=$DOCKER_PASSWORD \
  --docker-email=$DOCKER_EMAIL \
  --docker-server=$DOCKER_SERVER2 \
  --namespace openfaas-fn
```

```
$ kubectl edit serviceaccount default -n openfaas-fn
...
imagePullSecrets:
- name: my-private-repo # for hub.chenleileicode.com
- name: my-private-repo2 # for hub.chenleileicode.com:9090
```

- 验证

```
$ faas-cli list --gateway openfaas.chenleileicode.com:8080
$ faas-cli invoke  hello-world  --gateway openfaas.chenleileicode.com:8080
```

- 此外

```
# generate command creates kubernetes CRD YAML file for functions
$ faas-cli generate --help
...
Examples:
faas-cli generate --api=openfaas.com/v1alpha2 --yaml stack.yml | kubectl apply  -f -
faas-cli generate --api=openfaas.com/v1alpha2 -f stack.yml
faas-cli generate --api=serving.knative.dev/v1alpha1 -f stack.yml
faas-cli generate --api=openfaas.com/v1alpha2 --namespace openfaas-fn -f stack.yml
faas-cli generate --api=openfaas.com/v1alpha2 -f stack.yml --tag branch -n openfaas-fn
```
