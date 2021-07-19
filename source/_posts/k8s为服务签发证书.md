---
layout: post
title: k8s为服务签发证书
date: 2021-07-19 17:21:41
tags: 
- k8s
categories:
- 开发
---
仅限于集群服务制作证书。

<!-- more -->

#### 制作证书脚本
注意：仅限于集群服务制作证书，必须填入参数。参数1为文件夹名，参数2为 集群服务 dogbrother.prj-lee-sin.svc
```sh
mkdir $1
cd $1

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "server": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "ST": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

cat > server-csr.json <<EOF
{
  "CN": "admission",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
        "C": "CN",
        "L": "BeiJing",
        "ST": "BeiJing",
        "O": "k8s",
        "OU": "System"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -hostname=$2 -profile=server server-csr.json | cfssljson -bare server

echo "========================= ca.pem ================================"
cat ca.pem
echo "========================= server.pem ================================"
cat server.pem
echo "========================= server-key.pem ================================"
cat server-key.pem
```