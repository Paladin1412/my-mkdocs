```
# 查看证书 过期时间
$ cd /etc/kubernetes/pki
$ openssl x509 -in apiserver.crt -text -noout

Validity
  Not Before: Feb  7 12:25:44 2020 GMT
  Not After : Feb  6 12:25:44 2021 GMT
```

> 发现：时间有效期为一年。
>
> 现在要延长有效期。

- 方案1 kubeadm 命令手动更新。

```
$ cd /workspace/kubernetes/configs/
$ kubeadm alpha certs renew all --config=./kubeadm-config.yaml
```

- 方案2 go语言重新编译 kubeadm 修改为10年。

> https://github.com/kubernetes/kubeadm.git

