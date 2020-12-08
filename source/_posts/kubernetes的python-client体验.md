---
title: kubernetes的python-client体验
date: 2020-12-01 13:56:09
tags:
- kubernetes
- python
categories:
- python
- kubernetes
---
kubernetes提供了丰富的API接口，用户不仅可以通过CLI命令行去调用API接口，而且还可以通过client库方便的调用API接口

https://kubernetes.io/docs/reference/using-api/client-libraries/，提供了官方支持的和社区支持的不同语言的client库


本文将着重体验python版本的client库的使用

## 源码安装
```
git clone --recursive https://github.com/kubernetes-client/python.git
cd python
python setup.py install
```

## pip安装
```
pip install kubernetes
```

## Example

```
from kubernetes import client, config

# Configs can be set in Configuration class directly or using helper utility
config.load_kube_config()

v1 = client.CoreV1Api()
print("Listing pods with their IPs:")
ret = v1.list_pod_for_all_namespaces(watch=False)
for i in ret.items:
    print("%s\t%s\t%s" % (i.status.pod_ip, i.metadata.namespace, i.metadata.name))
```

```
root@ubuntu:~/python# python test_list_pods.py
Listing pods with their IPs:
10.244.0.8      default edgex-app-service-configurable-rules-d66d779c7-ksw7c
10.244.0.9      default edgex-core-command-6f7cd7d57f-tnmmp
10.244.0.18     default edgex-core-consul-64b88766-7q7kr
10.244.0.10     default edgex-core-data-57b99d7f89-gpfgs
10.244.0.11     default edgex-core-metadata-58dcc95ff4-9ssjg
10.244.0.12     default edgex-device-rest-7944449548-nzcfj
10.244.0.13     default edgex-device-virtual-6c7b8d7499-csz85
10.244.0.14     default edgex-redis-67fffb7666-pvqgw
10.244.0.15     default edgex-support-notifications-58dc7f76b4-tsdxn
10.244.0.16     default edgex-support-rulesengine-8657b7c988-zr8pv
10.244.0.17     default edgex-support-scheduler-d6fd467db-tfqw7
10.244.0.19     default edgex-sys-mgmt-agent-79b476d8dc-2jdw4
10.244.0.20     default edgex-ui-8cfc7f95d-vvhzc
10.244.0.4      kube-system     coredns-f9fd979d6-cqpss
10.244.0.3      kube-system     coredns-f9fd979d6-kvbzm
172.18.0.2      kube-system     etcd-kind-control-plane
172.18.0.2      kube-system     kindnet-br2qc
172.18.0.2      kube-system     kube-apiserver-kind-control-plane
172.18.0.2      kube-system     kube-controller-manager-kind-control-plane
172.18.0.2      kube-system     kube-proxy-4chlm
172.18.0.2      kube-system     kube-scheduler-kind-control-plane
10.244.0.7      kube-system     metrics-server-8dc97c749-h6w8n
10.244.0.6      kubernetes-dashboard    dashboard-metrics-scraper-7b59f7d4df-5rwc6
10.244.0.5      kubernetes-dashboard    kubernetes-dashboard-665f4c5ff-l6tqn
10.244.0.2      local-path-storage      local-path-provisioner-78776bfc44-xl4ht
root@ubuntu:~/python#
root@ubuntu:~/python# kubectl get pods --all-namespaces -o wide
NAMESPACE              NAME                                                   READY   STATUS             RESTARTS   AGE    IP            NODE                 NOMINATED NODE   READINESS GATES
default                edgex-app-service-configurable-rules-d66d779c7-ksw7c   1/1     Running            14         21h    10.244.0.8    kind-control-plane   <none>           <none>
default                edgex-core-command-6f7cd7d57f-tnmmp                    1/1     Running            21         21h    10.244.0.9    kind-control-plane   <none>           <none>
default                edgex-core-consul-64b88766-7q7kr                       1/1     Running            0          21h    10.244.0.18   kind-control-plane   <none>           <none>
default                edgex-core-data-57b99d7f89-gpfgs                       1/1     Running            19         21h    10.244.0.10   kind-control-plane   <none>           <none>
default                edgex-core-metadata-58dcc95ff4-9ssjg                   1/1     Running            21         21h    10.244.0.11   kind-control-plane   <none>           <none>
default                edgex-device-rest-7944449548-nzcfj                     1/1     Running            15         21h    10.244.0.12   kind-control-plane   <none>           <none>
default                edgex-device-virtual-6c7b8d7499-csz85                  1/1     Running            13         21h    10.244.0.13   kind-control-plane   <none>           <none>
default                edgex-redis-67fffb7666-pvqgw                           1/1     Running            0          21h    10.244.0.14   kind-control-plane   <none>           <none>
default                edgex-support-notifications-58dc7f76b4-tsdxn           1/1     Running            13         21h    10.244.0.15   kind-control-plane   <none>           <none>
default                edgex-support-rulesengine-8657b7c988-zr8pv             1/1     Running            0          21h    10.244.0.16   kind-control-plane   <none>           <none>
default                edgex-support-scheduler-d6fd467db-tfqw7                1/1     Running            14         21h    10.244.0.17   kind-control-plane   <none>           <none>
default                edgex-sys-mgmt-agent-79b476d8dc-2jdw4                  1/1     Running            0          21h    10.244.0.19   kind-control-plane   <none>           <none>
default                edgex-ui-8cfc7f95d-vvhzc                               0/1     ImagePullBackOff   0          21h    10.244.0.20   kind-control-plane   <none>           <none>
kube-system            coredns-f9fd979d6-cqpss                                1/1     Running            0          4d6h   10.244.0.4    kind-control-plane   <none>           <none>
kube-system            coredns-f9fd979d6-kvbzm                                1/1     Running            0          4d6h   10.244.0.3    kind-control-plane   <none>           <none>
kube-system            etcd-kind-control-plane                                1/1     Running            0          4d6h   172.18.0.2    kind-control-plane   <none>           <none>
kube-system            kindnet-br2qc                                          1/1     Running            0          4d6h   172.18.0.2    kind-control-plane   <none>           <none>
kube-system            kube-apiserver-kind-control-plane                      1/1     Running            0          23h    172.18.0.2    kind-control-plane   <none>           <none>
kube-system            kube-controller-manager-kind-control-plane             1/1     Running            1          4d6h   172.18.0.2    kind-control-plane   <none>           <none>
kube-system            kube-proxy-4chlm                                       1/1     Running            0          4d6h   172.18.0.2    kind-control-plane   <none>           <none>
kube-system            kube-scheduler-kind-control-plane                      1/1     Running            1          4d6h   172.18.0.2    kind-control-plane   <none>           <none>
kube-system            metrics-server-8dc97c749-h6w8n                         1/1     Running            0          21h    10.244.0.7    kind-control-plane   <none>           <none>
kubernetes-dashboard   dashboard-metrics-scraper-7b59f7d4df-5rwc6             1/1     Running            0          21h    10.244.0.6    kind-control-plane   <none>           <none>
kubernetes-dashboard   kubernetes-dashboard-665f4c5ff-l6tqn                   1/1     Running            0          21h    10.244.0.5    kind-control-plane   <none>           <none>
local-path-storage     local-path-provisioner-78776bfc44-xl4ht                1/1     Running            1          4d6h   10.244.0.2    kind-control-plane   <none>           <none>
```

更多的examples：https://github.com/kubernetes-client/python/tree/master/example
