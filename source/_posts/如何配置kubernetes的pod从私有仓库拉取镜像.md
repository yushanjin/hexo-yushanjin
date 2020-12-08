---
title: 如何配置kubernetes的pod从私有仓库拉取镜像
date: 2020-12-01 15:30:19
tags:
- docker
- kubernetes
categories:
- registry
- docker
- kubernetes
---

在实际使用中，用户往往搭建了自己的私有镜像仓库。kubernetes用户创建pod的过程中，如何从私有镜像仓库拉取容器镜像？

本文重点要介绍：如何让使用secret从私有的 Docker 镜像仓库或代码仓库拉取镜像来创建 Pod


# 前提条件：
   1. kubernetes集群

   2. docker私有镜像仓库或者docker hub上有Docker ID（这里使用Docker ID来演示）

创建secret有两种方法：

   1. 使用config.json文件

   2. 直接使用用户名+密码

# 生成config.json文件
```
root@ubuntu:~# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: shayu
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```
根据提示，输入用户名及密码。

用户名及密码会被存储到本地文件config.json中。

```
root@ubuntu:~# cat /root/.docker/config.json
{
        "auths": {
                "https://index.docker.io/v1/": {
                        "auth": "c2hheXU6MDI4Mxxxxxxx4="
                }
        },
        "HttpHeaders": {
                "User-Agent": "Docker-Client/19.03.13 (linux)"
        }
}

root@ubuntu:~# echo ""c2hheXU6MDI4Mxxxxxxx4=" |base64 -d
shayu:xxxxxxx
```

# 创建secret

方法一：

```
root@ubuntu:~# kubectl create secret generic regcred --from-file=.dockerconfigjson=.docker/config.json --type=kubernetes.io/dockerconfigjson
secret/regcred created
root@ubuntu:~#
root@ubuntu:~# kubectl get secret regcred -o yaml
apiVersion: v1
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOiB7CgkWFkZXJzIjogewoJCSJVc2VyLUFnZW50IjogIkRvY2tlci1DbGllbnQvMTkuMDMuMTMgKGxpbnV4KSIKCX0KfQ==
kind: Secret
metadata:
  creationTimestamp: "2020-12-01T07:09:19Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:.dockerconfigjson: {}
      f:type: {}
    manager: kubectl-create
    operation: Update
    time: "2020-12-01T07:09:19Z"
  name: regcred
  namespace: default
  resourceVersion: "974703"
  selfLink: /api/v1/namespaces/default/secrets/regcred
  uid: 96ed63a0-90d8-47e3-a1d9-aa841a9d1813
type: kubernetes.io/dockerconfigjson

root@ubuntu:~# kubectl get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
{
        "auths": {
                "https://index.docker.io/v1/": {
                        "auth": "c2hheXU6MDI4Mxxxxxxxxxx4="
                }
        },
        "HttpHeaders": {
                "User-Agent": "Docker-Client/19.03.13 (linux)"
        }
}
```

方法二：

```
root@ubuntu:~# kubectl create secret docker-registry regcred1 --docker-username=shayu --docker-password=xxxxxx --docker-email=xxxxxxx@163.com
secret/regcred1 created

其中:
regcred1: 指定密钥的键名称, 可自行定义
--docker-server: 指定docker仓库地址
--docker-username: 指定docker仓库账号
--docker-password: 指定docker仓库密码
--docker-email: 指定邮件地址

root@ubuntu:~#kubectl get secret regcred1 -o yaml
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJodHRwczovL2luZGV4LmRvY2tlci5pbxxxxxxxxxxxxxFpbCI6Inl1c2hhbmppbjA3NjdAMTYzLmNvbSIsImF1dGgiOiJjMmhoZVhVNk1ESTRNVEV3ZVhOcUxDND0ifX19
kind: Secret
metadata:
  creationTimestamp: "2020-12-01T07:26:30Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:.dockerconfigjson: {}
      f:type: {}
    manager: kubectl-create
    operation: Update
    time: "2020-12-01T07:26:30Z"
  name: regcred1
  namespace: default
  resourceVersion: "977436"
  selfLink: /api/v1/namespaces/default/secrets/regcred1
  uid: b552c5ac-92eb-424e-bf4b-2a2c281cb75a
type: kubernetes.io/dockerconfigjson

root@ubuntu:~# kubectl get secret regcred1 --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
{"auths":{"https://index.docker.io/v1/":{"username":"shayu","password":"xxxxxxx","email":"xxxxxxxx@163.com","auth":"c2hheXU6MDI4xxxxxxxxx4="}}}

```

# pod指定imagePullSecrets为上述创建的secret
```
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: xxxxxxx:yyyy
  imagePullSecrets:
  - name: regcred
```

参考：

(1) https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry

(2) https://kubernetes.io/docs/concepts/containers/images/
