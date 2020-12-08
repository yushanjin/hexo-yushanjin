---
title: kind快速部署Kubernetes环境
date: 2020-11-26 16:56:12
tags:
- kubernetes
- 云原生
categories:
- 云原生
---

# 什么是kind

kind：Kubernetes In Docker，顾名思义，就是将kubernetes所需要的所有组件，全部部署在一个docker容器中，是一套开箱即用的kubernetes环境搭建方案。使用kind搭建的集群无法在生产中使用，但是如果你只是想在本地测试或者开发使用，不想占用太多的资源，那么使用kind是不错的选择。同样，kind还可以很方便的帮你本地的kubernetes源代码打成对应的镜像，方便测试。

GitHub: https://github.com/kubernetes-sigs/kind

Documentation: https://kind.sigs.k8s.io/

# 安装kind

以Linux下安装为例：

```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.9.0/kind-linux-amd64
chmod +x ./kind
mv ./kind /usr/local/bin/kind
```

# 创建、查询集群

```
kind create cluster
```
该命令将默认创建名为kind的集群

```
root@ubuntu:~# kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.19.1) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂

root@ubuntu:~# kind get clusters
kind
```

```
kind create cluster --name test
```
该命令将创建名为test的集群

```
root@ubuntu:~# kind create cluster --name test
Creating cluster "test" ...
 ✓ Ensuring node image (kindest/node:v1.19.1) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-test"
You can now use your cluster with:

kubectl cluster-info --context kind-test

Thanks for using kind! 😊

root@ubuntu:~# kind get clusters
kind
test

root@ubuntu:~# kubectl cluster-info --context kind-test
Kubernetes master is running at https://127.0.0.1:44543
KubeDNS is running at https://127.0.0.1:44543/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

kubectl安装：https://kubernetes.io/docs/tasks/tools/install-kubectl/


查询集群节点
```
root@ubuntu:~# kubectl get nodes --context kind-test
NAME                 STATUS   ROLES    AGE     VERSION
test-control-plane   Ready    master   4m55s   v1.19.1

root@ubuntu:~# kubectl get nodes --context kind-kind
NAME                 STATUS   ROLES    AGE     VERSION
kind-control-plane   Ready    master   9m14s   v1.19.1
```

```
root@ubuntu:~# docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                       NAMES
7fa7f8313453        kindest/node:v1.19.1   "/usr/local/bin/entr…"   16 minutes ago      Up 16 minutes       127.0.0.1:44543->6443/tcp   test-control-plane
6fee527273e2        kindest/node:v1.19.1   "/usr/local/bin/entr…"   20 minutes ago      Up 20 minutes       127.0.0.1:36585->6443/tcp   kind-control-plane
root@ubuntu:~# docker ps -a
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                       NAMES
7fa7f8313453        kindest/node:v1.19.1   "/usr/local/bin/entr…"   16 minutes ago      Up 16 minutes       127.0.0.1:44543->6443/tcp   test-control-plane
6fee527273e2        kindest/node:v1.19.1   "/usr/local/bin/entr…"   20 minutes ago      Up 20 minutes       127.0.0.1:36585->6443/tcp   kind-control-plane
```
test-control-plane、kind-control-plane是运行在容器内的kubernetes节点，也正是kubernetes in Docker的意思

查询集群内运行的pod
```
root@ubuntu:~# kubectl get pods --all-namespaces --context kind-kind
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
kube-system          coredns-f9fd979d6-cqpss                      1/1     Running   0          22m
kube-system          coredns-f9fd979d6-kvbzm                      1/1     Running   0          22m
kube-system          etcd-kind-control-plane                      1/1     Running   0          22m
kube-system          kindnet-br2qc                                1/1     Running   0          22m
kube-system          kube-apiserver-kind-control-plane            1/1     Running   0          22m
kube-system          kube-controller-manager-kind-control-plane   1/1     Running   0          22m
kube-system          kube-proxy-4chlm                             1/1     Running   0          22m
kube-system          kube-scheduler-kind-control-plane            1/1     Running   0          22m
local-path-storage   local-path-provisioner-78776bfc44-xl4ht      1/1     Running   0          22m

root@ubuntu:~# kubectl get pods --all-namespaces --context kind-test
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
kube-system          coredns-f9fd979d6-f7wsz                      1/1     Running   0          18m
kube-system          coredns-f9fd979d6-j9xtx                      1/1     Running   0          18m
kube-system          etcd-test-control-plane                      1/1     Running   0          18m
kube-system          kindnet-jq867                                1/1     Running   0          18m
kube-system          kube-apiserver-test-control-plane            1/1     Running   0          18m
kube-system          kube-controller-manager-test-control-plane   1/1     Running   0          18m
kube-system          kube-proxy-swn5j                             1/1     Running   0          18m
kube-system          kube-scheduler-test-control-plane            1/1     Running   0          18m
local-path-storage   local-path-provisioner-78776bfc44-nwqjq      1/1     Running   0          18m
```

查询pod使用的镜像
```
root@ubuntu:~# docker exec -it kind-control-plane crictl images
IMAGE                                      TAG                  IMAGE ID            SIZE
docker.io/kindest/kindnetd                 v20200725-4d6bea59   b77790820d015       119MB
docker.io/rancher/local-path-provisioner   v0.0.14              e422121c9c5f9       42MB
k8s.gcr.io/build-image/debian-base         v2.1.0               c7c6c86897b63       53.9MB
k8s.gcr.io/coredns                         1.7.0                bfe3a36ebd252       45.4MB
k8s.gcr.io/etcd                            3.4.13-0             0369cf4303ffd       255MB
k8s.gcr.io/kube-apiserver                  v1.19.1              8cba89a89aaa8       95MB
k8s.gcr.io/kube-controller-manager         v1.19.1              7dafbafe72c90       84.1MB
k8s.gcr.io/kube-proxy                      v1.19.1              47e289e332426       136MB
k8s.gcr.io/kube-scheduler                  v1.19.1              4d648fc900179       65.1MB
k8s.gcr.io/pause                           3.3                  0184c1613d929       686kB
root@ubuntu:~# docker exec -it kind-control-plane crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID
1c881f7f0d306       bfe3a36ebd252       25 minutes ago      Running             coredns                   0                   75e161b37ab2d
c64dd6bb666f3       bfe3a36ebd252       25 minutes ago      Running             coredns                   0                   a8305dc572af5
fea2b69f5472d       e422121c9c5f9       26 minutes ago      Running             local-path-provisioner    0                   fb704b6340b63
52f0995ba00f8       47e289e332426       26 minutes ago      Running             kube-proxy                0                   4c787e616d5a7
b87cdcc514f59       b77790820d015       26 minutes ago      Running             kindnet-cni               0                   14eb70f9ca549
ce2c4e5b2b57f       0369cf4303ffd       27 minutes ago      Running             etcd                      0                   744a99a558714
93b5084a29992       8cba89a89aaa8       27 minutes ago      Running             kube-apiserver            0                   91f88afc5a39b
8b9579313058f       7dafbafe72c90       27 minutes ago      Running             kube-controller-manager   0                   8d54fdffef86e
10fbb8244ad3b       4d648fc900179       27 minutes ago      Running             kube-scheduler            0                   a985ae4a105bc
```
其中，crictl命令可以理解为docker命令


# 删除集群

```
kind delete cluster --name test
```
删除名为test的集群，--name未指定的话，将默认删除kind集群


# 创建高可用kubernetes集群
```
root@ubuntu:~# cat kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: control-plane
- role: control-plane
- role: worker
- role: worker
- role: worker

root@ubuntu:~# kind create cluster --config kind-config.yaml --name test2
Creating cluster "test2" ...
 ✓ Ensuring node image (kindest/node:v1.19.1) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦 📦 📦
 ✓ Configuring the external load balancer ⚖️
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining more control-plane nodes 🎮
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-test2"
You can now use your cluster with:

kubectl cluster-info --context kind-test2

Have a nice day! 👋

root@ubuntu:~# kubectl get nodes --context kind-test2
NAME                   STATUS   ROLES    AGE     VERSION
test2-control-plane    Ready    master   5m12s   v1.19.1
test2-control-plane2   Ready    master   4m38s   v1.19.1
test2-control-plane3   Ready    master   3m22s   v1.19.1
test2-worker           Ready    <none>   2m5s    v1.19.1
test2-worker2          Ready    <none>   2m5s    v1.19.1
test2-worker3          Ready    <none>   2m5s    v1.19.1

root@ubuntu:~# kubectl get pods --all-namespaces --context kind-test2
NAMESPACE            NAME                                           READY   STATUS    RESTARTS   AGE
kube-system          coredns-f9fd979d6-jbpcz                        1/1     Running   0          5m16s
kube-system          coredns-f9fd979d6-ng4qp                        1/1     Running   0          5m16s
kube-system          etcd-test2-control-plane                       1/1     Running   0          5m15s
kube-system          etcd-test2-control-plane2                      1/1     Running   0          4m54s
kube-system          etcd-test2-control-plane3                      1/1     Running   0          2m59s
kube-system          kindnet-gxpjn                                  1/1     Running   0          4m55s
kube-system          kindnet-jqnx5                                  1/1     Running   0          2m20s
kube-system          kindnet-lczmx                                  1/1     Running   0          2m18s
kube-system          kindnet-q8bcn                                  1/1     Running   0          2m19s
kube-system          kindnet-q9ng2                                  1/1     Running   0          3m37s
kube-system          kindnet-s7kfb                                  1/1     Running   0          5m14s
kube-system          kube-apiserver-test2-control-plane             1/1     Running   0          5m15s
kube-system          kube-apiserver-test2-control-plane2            1/1     Running   0          4m54s
kube-system          kube-apiserver-test2-control-plane3            1/1     Running   1          3m9s
kube-system          kube-controller-manager-test2-control-plane    1/1     Running   2          5m14s
kube-system          kube-controller-manager-test2-control-plane2   1/1     Running   0          4m54s
kube-system          kube-controller-manager-test2-control-plane3   1/1     Running   0          2m8s
kube-system          kube-proxy-47nc7                               1/1     Running   0          5m16s
kube-system          kube-proxy-5799m                               1/1     Running   0          4m55s
kube-system          kube-proxy-cvm49                               1/1     Running   0          2m18s
kube-system          kube-proxy-s7rsp                               1/1     Running   0          2m18s
kube-system          kube-proxy-sxwgl                               1/1     Running   0          3m37s
kube-system          kube-proxy-wvskh                               1/1     Running   0          2m20s
kube-system          kube-scheduler-test2-control-plane             0/1     Running   2          5m15s
kube-system          kube-scheduler-test2-control-plane2            1/1     Running   0          4m54s
kube-system          kube-scheduler-test2-control-plane3            1/1     Running   0          2m31s
local-path-storage   local-path-provisioner-78776bfc44-fkdwq        1/1     Running   1          5m12s
```

上述过程，我们创建了两个kubernetes集群，kind和test2；可以看到我们在使用kubectl访问集群时，增加了参数：--context，这在本地的配置文件里指定了
```
root@ubuntu:~# cat .kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01URXlOekF3TkRJeU9Wb1hEVE13TVRFeU5UQXdOREl5T1Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTERTCm5TU1NHS05PUUlCVWpQeXpYOFRDZm5IamExV2JxeXZJeHV3Y1Aybmh6Zi9EdG9wVDRnSTVjMlBMV3Qrd05BUFUKUHA2dGV3ZkhNQ3N6MnJLbnhaRmtWS2c5NXVFdW53V1ZuZnlmeGV0TjlOU1JTZ2s2dkJuVUt6SFliRXIybEY0LwpicFVxT2IzWnkxQXdYNlpyRTN3Y1I1RjdLV2trT0FZbHdobUtiLzIwOVZJRG4yMW9CMHMzNXgrM3Z2L2gzQ3VaCkJCQjJnNVBKMm4xc1pwd05scnZDMmh0RmJDSjQwQVNXZmNsUksyejBYUEIvdzdNQlBXMHp1cnpEMUVBcGdZVUcKUGUwOEx3dW1zMllZR3Y5TDJXMERhWW90c2JoVXlWZWRDNnpjeWtUZ0hOb1B6LytvZlBHQmxYTko3S2lwd2Z6ZQo2dzdHZGJhbjlpRms4cjAyaVQ4Q0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZEY2liT2RWZTN4U0w0RmMxOWVJdmZnSTBHcjZNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCS1lKSVFaenhSblFWWWp4TnlTaXJualFvZUM3OGtDMERNYjZpRFBvMFdEbmdTUjhRYgpqRjZxR2ZPMTFTcDNjeVl3V1IrM3UzU2pzQUJUUG5jcWZpWnEyN3VDMlRYNjdMQzRPNVFoVkdlRlBDdXJNNHluCi8weDZuOUZrMko5YmpTT0o1aDB2NmQ1eXBibk5sNVgvN1czWFVyK2tjOSswWG1sMFN0dmJVZ0hCaWVmWGxtZ3AKUXl5TlFtNU1yWlRFcWJOT0JubW1RWUJYWDdydWVLZXhVYUJ4QXQrRVJVWHBVOWNhbkNWWWhuZFBuMVNBYVladApsQkZyamRSNzg1SE1EV21qTW5UalJXSUhOSWd2NUgwKyt2MmN5cjRSK1lUWGo3S0JrdEZTRDA2U0I0RDZVeFk2CmRUZ25UMngzRXJvSERPZURjWWRrVmt2TkF5aC9JUGFTd2R6cwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://127.0.0.1:36585
  name: kind-kind
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01URXlOekF6TWpJeE9Gb1hEVE13TVRFeU5UQXpNakl4T0Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSi85CjlZQ0VseTNRLzRuc2t5dGZZY2Zpby9jcDJoaSsxRWpPOGMrVmF4YTkrbTIrdldBT1VhU0xQZnQ0VmRSL1hIWjYKT3FQSEY5K3ozNEZIb1p4RTJIY3kxd3R0Y1hOTXk1Qy9VMUdnakgramZLaWNmbVJmS3luS1M4SU80T3pLRHBKUQpodHQzb0JxeDVHdU0wSUpkVmsydVg5RjhmVHEyTllaN2lzK0NOVWdXM2dxMXA4SzNkZWxjaDYyM2NBSHhSS0JLCmZEZC9iZnZiZitvR0ZQS29BNWhLcXVKb3BDVFcrN1VZdnA4ZCs0QTgwYkZ2cG5CSmlUK1pGU0sxRExKZHBuRlgKV0pDOHdwNzNBYU9KSHAydnNTZ0p6ajFvUHUvL1lnSWNPNStoSGl6bThVcnVGQWxkWmE2ZzN4VnFCazNDZnducApsaU50MzdiblJrZ3VUNUU5cFFNQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZNdnpFOXJORVFUWGNyUmJHcllmQ0J6VEpZby9NQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFBMjJZS1htS29kTkVmUklGS1ZOMXk1alRjQ2FIVnAyd3QvR1RNT3dHU1pYQXhlN2xBOQpIZDVCWHJoaXB6bkNPR1hxakRoZkVkaWVIa3VlcDNQWGxVTWxxa20yUGZkdUp0cTJ3T0piR0ZTVWZRR2xpS09MCkJ0eDRwVmd1dC9EQW11ZHNNNFlSeGRTS1R3bkppcjBBbkNnSWprd0gvZzdzekFYTVppMmZkYk9oL3NzcVFlZU4KREYrc01ld0czK2pSaVVqTEM2ck9sb0UySzArVjNhMDlMbmNiYlRCNDBVM0pZR0VvallzVExEdXZPakhuZElkKwpyM0FHMlNXMDRYek8wcHF4VDhlRjlkUE5qVG83SlpIeDc3RUtuNVNYU1YweGRuTTR1eHVBWDBoVExLenFyMlRNCmVJTzlZNGZWeHdCQk9BaFBQVmFTMW5SZy9GMXFIWWNKK20wUgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://127.0.0.1:34927
  name: kind-test2
contexts:
- context:
    cluster: kind-kind
    user: kind-kind
  name: kind-kind
- context:
    cluster: kind-test2
    user: kind-test2
  name: kind-test2
current-context: kind-test2
kind: Config
preferences: {}
users:
- name: kind-kind
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURFekNDQWZ1Z0F3SUJBZ0lJTHRQZkozeVliVHN3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURFeE1qY3dNRFF5TWpsYUZ3MHlNVEV4TWpjd01EUXlNekZhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXJEaURMOWhSUGduVzNwdlcKQ1pNekVlTG0vRTJuVmlXdG9icTVoZUdrdG1Udi9aWEUydG9uKy96ZWROOThvM3J4ZEdneDNTZDRndjZtSnpQbgpFZ3hTRDcrQnErSmt5WG5oQThPM1RQakN0aHdES1JIZjEzbkI4NDNlbjZ3S3E5K3A1RHB2UFNNSkgyQkJLckkrCjlHWTFvTWp2SnRWSjMyZ1dhTnlJV0hzTkE0QmdPM0RRT2Y2cGN6bXZhbllKZFliYTFpQmpKUEVaRURPNEpFdHIKZWYrTUFHb1diSFlaeDdySWo5eTJoNzdOQi9DV0k1alVSUXp6MWQ0ZlFnV05WR1cyblVXbGlGZjlEWlRVR25CMwpHMHBUTjc5N1U0bUNYdWVBUjFxY25idS9YNjBMb2l6bEpKVWJzcjZYUWFaRmtKS2FzSGx2Tmdydm11cWhnSjZTCmxCWWNIUUlEQVFBQm8wZ3dSakFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0h3WURWUjBqQkJnd0ZvQVVOeUpzNTFWN2ZGSXZnVnpYMTRpOStBalFhdm93RFFZSktvWklodmNOQVFFTApCUUFEZ2dFQkFBay8xbjJqLzhGdlhISm94V0Q3dUl4WUxTUmlrTjcxWWdOOW1yckVYRjVFUkxBMjI5eEtyT1c1ClV2Tm9nZGlqdWpEbDhoVjRoU1hkY3M0dE9ZNmZYbExkN3JhN25vL054UUtHM1lPdktWM2g0eS9wUEYxMENQL0sKTTFhSnNkY1U2aG8wbnZrL1dQSDB2ckxtNE1jUHpBbFUvazdvd3FiemcybHZQdDBCMTJjZHh4bk44UHp5VU4zZwpQQVBzT3hab1hwQnVpNjkxcEt2a1VDUm1MY0dUTHowT0Y5YXVOaUhRRG1qMG1hSEg2ckI2c0kvQzBuZ08vaCtSCmNsUnBGYk9uQnZRbW1nRjZER0E4cGxUSUM2ZE1JOGtXRzVaQWFSbzMrblFLME5sMERaUDVBTjM1cE1yS0M1R3cKeTdHaVlRQkFCVlJOZ2FJemg1bktHUXFJVVFXRUU5TT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBckRpREw5aFJQZ25XM3B2V0NaTXpFZUxtL0UyblZpV3RvYnE1aGVHa3RtVHYvWlhFCjJ0b24rL3plZE45OG8zcnhkR2d4M1NkNGd2Nm1KelBuRWd4U0Q3K0JxK0preVhuaEE4TzNUUGpDdGh3REtSSGYKMTNuQjg0M2VuNndLcTkrcDVEcHZQU01KSDJCQktySSs5R1kxb01qdkp0VkozMmdXYU55SVdIc05BNEJnTzNEUQpPZjZwY3ptdmFuWUpkWWJhMWlCakpQRVpFRE80SkV0cmVmK01BR29XYkhZWng3cklqOXkyaDc3TkIvQ1dJNWpVClJRenoxZDRmUWdXTlZHVzJuVVdsaUZmOURaVFVHbkIzRzBwVE43OTdVNG1DWHVlQVIxcWNuYnUvWDYwTG9pemwKSkpVYnNyNlhRYVpGa0pLYXNIbHZOZ3J2bXVxaGdKNlNsQlljSFFJREFRQUJBb0lCQVFDYlJzUzVVYTlHWVRhegpOUXhSUzcvREE3TEJqdjR1QlFDOURnOFJyL1dEWWhTanJmSjBaRGVpMGtaOFY3Z1g2ZFJqNFVIOEpRZGFER0VnCmZZSjhXbEZ1MDNzRno3U1JsMnNTcXRiTTlva1FDc2Vxc3V3QWFrNDkydzc3SmZIbEwxOE5ZTVpFK0I3VWhFT2QKVEdMSWxwTUpxY0UrWVJZZThNa3J1SkxTTy9mcXk4eW1zWnk5ZHlyOWIvTjJnZm1iYW1kWVpVa3hneFVaVVB6RQpqdDBFZEZ5YThHK29YcUlMakZoR3oyQnBHSFJIVnNJYjlGdEhEbktzTVV5YzNGQXZyTmVjTHYyYmw1a0tjb1pBClY3ay9pMHhiVGpWK0tVSXQ2STlPb09aNkIzNjZkbDJjUHNQQmJtK3pRN3A4a0l0SXdoanpOUVN3QytjVFhVYnIKQ2Yxb3BpYjlBb0dCQU11RlNJVlpjaVk1akNSWlRnd0dEczlybGdsWW9hNFA4TkUyMng5RFdoMVNyS2k2and0VgorcGphM1V3ZTBYRHBWTFIrZXNKZ1NYZlBqSk5WVW5KS0RKeElBTGNHM01qek9RcUlwYjMxMXBoenJsdWo4WGllCi8wQ0ZMU0YyR2N0b1B2cUxPUWFvQjNFcy96UmxzUmVOSlMvUEp0NG9pS29ERmI3d05zbFRYVWRqQW9HQkFOaWgKRXhadG5qd1RRZzBpUHU0SERYUlU0M0hzQ2pWMFh0MENGclY0Q2Qzcjc3ZUsxd2RtSkhVUWczV0VKeUxBREhyVQpIa0EzKzRJdXdmcVE0d0FwZHF6SFF4UUtJbXA2dEpVTElYL2xhNk1RbFltSTREcHFVUWRETi8yN0V4bXAwRkZVCkc1b2FNb1BoOVNLMk0wZ2RkWjJpS3llZ2I5dkJLNWJORzVTdUhTWi9Bb0dCQUsyaVM4b0JFdU5MeTZXalQzUHcKb3lnUmlOTDJmQklONVk0SStBK0hIZFhRbUIvbjhteGdjVW1CeUxYTndUQk0wWWlnTThtcjdtSTZmNXVmZXBTcApXbkxtOXowdnJLUUE1bFIzV3JoamlpOU0ycCt5a2l3dnNtUHdleDJHTGVHZFVjWGRpOHlEQkw1by9sNU11RGI0Cm81WlRiTHl5NWszdURkcDJCTGZrMkxzekFvR0JBTEp6aGdqTXhqUFEzWEY2UzRMRFZvY0ZRdFBlME00V0RldGIKeEI4N1FrMkpCVkVhVTJacDh4Qm9TUkt1aVpxcnY5d1REdFJ5Q1lMRlI5QkVPR3N5dk9zNXZuMHNtQXRGQjZ0YgpudjMvbkxxWWQ4YnpkVnRKcDNRbklHR3BFT1BzS29wRWtmUlJMbG5MOHFia2xyd0tZSkE1UGZtSHhYMnUxRnlHCm0valBzWDI3QW44cStrMXBqRFlMakMzSjlXWkE2a3Blc05yMENaU0RaTjNhZy9LNUxzQTVaekhaclhMQm9EaEYKekJ5ankvVVpUM1RlOXVxM2dTb1pNdnp1TVRRYWVLclFmOFBHQi9sNUVvdDJxSkMza1piR1F5cXNQVG8yT0JSdApnNytzdjE2azRieEZlMkVRVmU4M0NXR0pneEZkKzc1c1NxZGxON1RKZXJVOUdxbnM1dmk1Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
- name: kind-test2
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURFekNDQWZ1Z0F3SUJBZ0lJR2toZno1Qy9Yc2d3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURFeE1qY3dNekl5TVRoYUZ3MHlNVEV4TWpjd016SXlNakJhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXJ6azZ2RFJsMjI4V1RndUEKYnNNN0tLVUVaaWRmbVBKVW1kaitmdCs1UTZFSHFvUitzOHVycDVVc0tOY0krZzdoYkNJa2VzdzE4OGZ0NDVlSwp4eGpmV1NiUXNiSjYyb3RQRUVoSGpPZElRTHVxQlBqR1J1N3NwUE8yRzFnMFZTMm9UY0VUVlYzQ2RCUFFPRjNOCm9WblhPdzQwSWttWXVURDZHUEdNT2VwN05Rak4vZERybVphMy94WWxJK2loVHZJbnZXNjVYYmd0NTNwS203YXgKOUNSVGhjNUtONjg2TzVLekR6ZnhPZGtpUUhJdzNXL3BpTXRvOVV5QUF3V2RoUmc0K0VacW5OTjZXREl4RlNPNQpJN0Z6c3dtT2s0L29wcW55ZHhmNnFYamF6MGlvTTNyM0I4bUY0c09VakhtaDJyUnB3a3ZpQUxkV3AvbVo0ek5uClJYYVp3UUlEQVFBQm8wZ3dSakFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0h3WURWUjBqQkJnd0ZvQVV5L01UMnMwUkJOZHl0RnNhdGg4SUhOTWxpajh3RFFZSktvWklodmNOQVFFTApCUUFEZ2dFQkFIMHg2am5uajNSQWxBY0dOZmxyY1Zob01XVUpOZCtVak5OSXNSTHRaSzJTYTh6SThCK0VIeUxFCndhRjJGMzZ0VTV6L1JiSFluK3d3Q3ByQVJjLzhYTUx2OEpnMVU2aDg2SkxpNm9qc29Vb0dhUFNoWmI3S1YyOUQKSE5JQ2p5M2QrenhjdEV4OWFTRDB3WkR0cFpVaS9tZlQ3MklRekVUazQ2QTQ4bldYdkx0MWJ6TEZ4T1hQZVB1WgpiZG1lcWR5QmN1TXI0MUppcEJVUlpOWDgxQ0xIQ1ZQU2tsRHkySVl0OUgxeVNmb3RHMUV4ZHlDZE1vRjZTVUdhCkpLY0psNGFSQ3JNLzNqSGx3YVpGcWR0WkFFZ2pzM3lOYXllTW9SYi93bEdpWkdaQTFhQ3JNZlU5bXRnTlMrMWIKbmE1M0RxVmRKZHBTSnhWSjNPQXpUY2s0OVV1Z2c4ND0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBcnprNnZEUmwyMjhXVGd1QWJzTTdLS1VFWmlkZm1QSlVtZGorZnQrNVE2RUhxb1IrCnM4dXJwNVVzS05jSStnN2hiQ0lrZXN3MTg4ZnQ0NWVLeHhqZldTYlFzYko2Mm90UEVFaEhqT2RJUUx1cUJQakcKUnU3c3BQTzJHMWcwVlMyb1RjRVRWVjNDZEJQUU9GM05vVm5YT3c0MElrbVl1VEQ2R1BHTU9lcDdOUWpOL2REcgptWmEzL3hZbEkraWhUdkludlc2NVhiZ3Q1M3BLbTdheDlDUlRoYzVLTjY4Nk81S3pEemZ4T2RraVFISXczVy9wCmlNdG85VXlBQXdXZGhSZzQrRVpxbk5ONldESXhGU081STdGenN3bU9rNC9vcHFueWR4ZjZxWGphejBpb00zcjMKQjhtRjRzT1VqSG1oMnJScHdrdmlBTGRXcC9tWjR6Tm5SWGFad1FJREFRQUJBb0lCQUR5a0tNQ2p2YkNRcEg2RQpHb0c2elVtR3Vwd0QrbUM3VlM0ZFhBNWFyUXBMdTVSMjRFYW5NUlFCVzFRUy80ZFRDUTdjVGhXMWdPS0tpYmpmClpHYjlJNmI5K1BIV25BL3djSDlwRkdJZVZQSWFRSUFSL01UbHdUNWhIZUFleVpYRkJGOU1kNzF1Z25LYnZNOFYKSDZvOHBuRkl2Q0ExcWtaRlBmak45OEsvZEw1b1o0U3BrRjVxT2lTZ1MzdjVvTVM5UzR4OGRLTERtNldlYlJIZApCNTByaXppamJkVFJqRVZ4Yjk4ZG9GK0pYbWhwYS9jSXg1WmNrOFoxVElCMUJvRmpaSytsM014S0UyZkdNYnBrCjNUMUk3cGlFNVg0M1N3YmRQUksvNGxKT1NYUjdUeHNkN3kwajFuMjhtaHUwdEJPdUlnbHUrYTZ1UW5mZ1lVOTQKNTN1SjhEVUNnWUVBd2tPY1crQzJvcm9yQU5MUld6TWVlZnlmNG1pelpTQU9uelJ5WGRtWU9paWRiU1lRWlRXegpOWjJrSWlWLzBwS0hxM213dWp4K1gyQlRqVGE5MGNRUnVndkdzcUhRVmdQOVczZ1V0TWtLTTVobG9sUHZwTUZvCnU1Z2JNMjNCM1luYWdWR0w3ajVaUmVLWEYrbkRWbjdwdUt4TmFzZ2tmL1dZYy9CenlCeWFBRk1DZ1lFQTV1aVMKUVJqbzFlSVlsK0dRSUY5Rzh3dmdxL1puZS9yVSt3ZWVzcSt6OHZEck43TlRJUnZMcllVY0NZaHRWY0VWVkF5UgpyZzkvQU50UDVIcEFoQ2Z3OHJkcjNCNmxqY0JDMjhVQ0JMOWFsRXpDbjVCRFBEVVN3Z0Y3SE5UaUwvUE9OWEFkCmlJc1pydHJibzlUdzkyOS9HRUQ1aFV5N21DR3J1d0Y1d1gxWEN4c0NnWUE1eENFYXNSZWVDLzM5b0xMZ2k3TGsKVTFxMzJLcC94NmlSYnVjVFFVRWpDakRGNUN1NzdOdjlkWUw1SkcxK0VGU0hpUWdrV1JpN0E4blVsQktkN2MvWApvdWpTOVlzZUNOR3VBV2NtMnlGTmRtUENnWE1oYXVIWjVzRXY2ZE5jTFVIc2NuTkp4UUNHNTNwR2doeXorOGxFClFQaEVhSDl5RFhYb0EvaHA2UmRpUVFLQmdGaWk3QWxyQzIyV3hjUC9oUGk0T2g3djcwVnpaNVB5M0RDa1l5bksKUW5RK1FMeDM3TEFuNEU1eWF5bkpvZGFxTUlxNzdHdjViTklpWFkraDBnUW81TmYyeXNPTFRCZVd0dE52MDIrSgpHTGNXcEJybUlMa0swbkdBYWdiT1BTa1ZHSkh3d0pWNmQ5aGtFSzNaL3Ntc2xnZjBZUlBuT1plVFRUMlN1bThvCnN2SURBb0dBWk85dWNVS215OHBmM1A5L2NZQmVOVU9EOUFSQytna29DQkE5V1Zuclk3bDk4ZFRuWjFFbzZZcEkKUEkvRmFZRWcvYnptT3k1bzRDNVc0NGpZZnJLMFh0aHZkN3lIbVJidnNRRzBSV21EZnQ5OE5EbU9MbG44a0RMQQpUb1ROOU5IbWJVWmQ1VXZLa3BMTWhLbWQ5M05hdklXajlnV245TXh4MXBaUURleWg0SUk9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
```
可以看到两个cluster，两个context。context指定了cluster及user，users字段列出了两个user

context名为kind-kind，cluser为kind-kind，user为kind-kind，意思就是当指定--context为kind-kind时，将使用kind-kind的user去访问cluster为kind-kind的集群。

current-context: kind-test2说明当前的context使用的是kind-test2，如果不指定--context，将默认使用kind-test2

当然，用户可以修改当然默认的current-context
```
root@ubuntu:~# kubectl get node
NAME                   STATUS   ROLES    AGE     VERSION
test2-control-plane    Ready    master   2d23h   v1.19.1
test2-control-plane2   Ready    master   2d23h   v1.19.1
test2-control-plane3   Ready    master   2d23h   v1.19.1
test2-worker           Ready    <none>   2d23h   v1.19.1
test2-worker2          Ready    <none>   2d23h   v1.19.1
test2-worker3          Ready    <none>   2d23h   v1.19.1

root@ubuntu:~# kubectl config
Modify kubeconfig files using subcommands like "kubectl config set current-context my-context"

 The loading order follows these rules:

  1.  If the --kubeconfig flag is set, then only that file is loaded. The flag may only be set once and no merging takes
place.
  2.  If $KUBECONFIG environment variable is set, then it is used as a list of paths (normal path delimiting rules for
your system). These paths are merged. When a value is modified, it is modified in the file that defines the stanza. When
a value is created, it is created in the first file that exists. If no files in the chain exist, then it creates the
last file in the list.
  3.  Otherwise, ${HOME}/.kube/config is used and no merging takes place.

Available Commands:
  current-context Displays the current-context
  delete-cluster  Delete the specified cluster from the kubeconfig
  delete-context  Delete the specified context from the kubeconfig
  get-clusters    Display clusters defined in the kubeconfig
  get-contexts    Describe one or many contexts
  rename-context  Renames a context from the kubeconfig file.
  set             Sets an individual value in a kubeconfig file
  set-cluster     Sets a cluster entry in kubeconfig
  set-context     Sets a context entry in kubeconfig
  set-credentials Sets a user entry in kubeconfig
  unset           Unsets an individual value in a kubeconfig file
  use-context     Sets the current-context in a kubeconfig file
  view            Display merged kubeconfig settings or a specified kubeconfig file

Usage:
  kubectl config SUBCOMMAND [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).

root@ubuntu:~# kubectl config use-context
Sets the current-context in a kubeconfig file

Aliases:
use-context, use

Examples:
  # Use the context for the minikube cluster
  kubectl config use-context minikube

Usage:
  kubectl config use-context CONTEXT_NAME [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
error: Unexpected args: []

root@ubuntu:~# kubectl config use-context kind-kind
Switched to context "kind-kind".

root@ubuntu:~# kubectl get node
NAME                 STATUS   ROLES    AGE    VERSION
kind-control-plane   Ready    master   3d1h   v1.19.1
```
实际上，kind内部创建集群过程中，也是使用kubeadm，从上述创建过程中的打印也似乎能猜到这一点

后面，就可以使用搭建好的集群环境，开始你的开发、测试工作了~~~
