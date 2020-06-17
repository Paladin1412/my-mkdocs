- ### Get `ofc-bootstrap`

```
$ git clone https://github.com/openfaas-incubator/ofc-bootstrap

$ curl -sLSf https://raw.githubusercontent.com/openfaas-incubator/ofc-bootstrap/master/get.sh | \
 sudo sh
```

- ###  Create your own `init.yaml`

```
$ cat ~/.docker/config.json

export SERVER="docker-registry.qiyi.virtual"
export DOCKER_USERNAME="xxxx"
export DOCKER_PASSWORD="xxxx"

kubectl create secret docker-registry registry-secret \
        --docker-username=$DOCKER_USERNAME \
        --docker-password=$DOCKER_PASSWORD \
        --docker-server=$SERVER \
        --dry-run -o jsonpath="{.data.\.dockerconfigjson}" | base64 -d > ~/.docker/config.json
```

```
# gitlab personal_access_tokens for OpenFaaS Cloud
ktb3vhnx3MhdZ8ofznsF
```

```
# 安装
$ ofc-bootstrap apply --file init.yaml
# 卸载
$ ./scripts/reset.sh
```

