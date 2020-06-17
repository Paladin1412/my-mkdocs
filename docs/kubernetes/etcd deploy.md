### ETCD  部署 【无证书】

- 安装

```
$ yum install -y etcd
```

-  配置

```
$ vi /etc/etcd/etcd.conf
```

```
# 1. etcd.conf  [on centos-master-01]
#[Member]
ETCD_NAME="etcd1"
#ETCD_NAME="etcd2"
ETCD_HOST="192.168.10.101"
#ETCD_HOST="192.168.10.102"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#
#[Clustering]
ETCD_INITIAL_CLUSTER="etcd1=https://192.168.10.101:2380,etcd2=https://192.168.10.102:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="existing"
#
#[Proxy]
#ETCD_PROXY="off"
#
#[Security]
ETCD_CERT_FILE="/etc/kubernetes/pki/etcd/etcd.pem"
ETCD_KEY_FILE="/etc/kubernetes/pki/etcd/etcd-key.pem"
ETCD_TRUSTED_CA_FILE="/etc/kubernetes/pki/etcd/ca.pem"
ETCD_PEER_CERT_FILE="/etc/kubernetes/pki/etcd/etcd.pem"
ETCD_PEER_KEY_FILE="/etc/kubernetes/pki/etcd/etcd-key.pem"
ETCD_PEER_TRUSTED_CA_FILE="/etc/kubernetes/pki/etcd/ca.pem"
#
#[Logging]
#ETCD_DEBUG="false"
#
#[Unsafe]
#ETCD_FORCE_NEW_CLUSTER="false"
#
#[Version]
#ETCD_VERSION="false"
#
#[Profiling]
#ETCD_ENABLE_PPROF="false"
#
#[Auth]
#ETCD_AUTH_TOKEN="simple" 
```

- 启动

```
$ systemctl daemon-reload 
$ systemctl start etcd 
$ systemctl status etcd
```

- 检查

```
$ etcdctl -version
$ export ETCDCTL_API=3
$ etcdctl member list
```



### ETCD  添加证书

```
$ cfssl print-defaults config > ca-config.json
$ cfssl print-defaults csr > ca-csr.json
```

```
cat > ca-config.json <<EOF
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "www": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            },
            "client": {
                "expiry": "8760h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "etcd": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
EOF
```

```
cat > ca-csr.json <<EOF
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ],
    "ca": {
       "expiry": "87600h"
    }
}
EOF
```

```
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

```
cat > etcd-csr.json <<EOF
{
    "CN": "etcd",
    "hosts": [
    	"localhost",
      	"127.0.0.1",
      	"192.168.10.101",
      	"192.168.10.102",
      	"centos-master-01",
      	"centos-master-02"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF
```

```
$  cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd etcd-csr.json | cfssljson -bare etcd 
```

```
$ vi /usr/lib/systemd/system/etcd.service
```

```
ExecStart=/usr/bin/etcd --data-dir=${ETCD_DATA_DIR} --name ${ETCD_NAME} \
  --initial-advertise-peer-urls https://${ETCD_HOST}:2380 --listen-peer-urls https://${ETCD_HOST}:2380 \
  --advertise-client-urls https://${ETCD_HOST}:2379 --listen-client-urls https://${ETCD_HOST}:2379,https://localhost:2379 \
  --initial-cluster ${ETCD_INITIAL_CLUSTER} \
  --initial-cluster-state ${ETCD_INITIAL_CLUSTER_STATE} --initial-cluster-token ${ETCD_INITIAL_CLUSTER_TOKEN} \
  --client-cert-auth \
  --ca-file=${ETCD_TRUSTED_CA_FILE} \
  --trusted-ca-file=${ETCD_TRUSTED_CA_FILE} \
  --cert-file=${ETCD_CERT_FILE} \
  --key-file=${ETCD_KEY_FILE} \
  --peer-client-cert-auth \
  --peer-trusted-ca-file=${ETCD_PEER_TRUSTED_CA_FILE} \
  --peer-cert-file=${ETCD_PEER_CERT_FILE} \
  --peer-key-file=${ETCD_PEER_KEY_FILE}
```

```
$ vi /etc/etcd/etcd.conf 
```

```
#[Security]
ETCD_CERT_FILE="/etc/kubernetes/pki/etcd/etcd.pem"
ETCD_KEY_FILE="/etc/kubernetes/pki/etcd/etcd-key.pem"
#ETCD_CLIENT_CERT_AUTH="false"
ETCD_TRUSTED_CA_FILE="/etc/kubernetes/pki/etcd/ca.pem"
#ETCD_AUTO_TLS="false"
ETCD_PEER_CERT_FILE="/etc/kubernetes/pki/etcd/etcd.pem"
ETCD_PEER_KEY_FILE="/etc/kubernetes/pki/etcd/etcd-key.pem"
#ETCD_PEER_CLIENT_CERT_AUTH="false"
ETCD_PEER_TRUSTED_CA_FILE="/etc/kubernetes/pki/etcd/ca.pem"
#ETCD_PEER_AUTO_TLS="false"
```

```
$ export ETCDCTL_API=3
$ etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.pem \
  --cert=/etc/kubernetes/pki/etcd/etcd.pem \
  --key=/etc/kubernetes/pki/etcd/etcd-key.pem \
   --write-out=table \
   member list
```

- Troubleshooting

```
# start request repeated too quickly for etcd.service
$ systemctl reset-failed etcd.service
$ systemctl stop etcd.service
$ journalctl -xe | grep etcd
```

