---
title: kubernetes部署httpd服务
date: 2020-12-02 15:08:32
tags:
- kubernetes
categories:
- kubernetes
---

```
root@ubuntu-001:~# cat httpd.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      run: httpd
  template:
    metadata:
      labels:
        run: httpd
    spec:
      containers:
      - name: httpd
        image: httpd
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  type: NodePort
  selector:
    run: httpd
  ports:
  - protocol: TCP
    nodePort: 30000
    port: 8080
    targetPort: 80

root@ubuntu-001:~# kubectl apply -f httpd.yaml
deployment.apps/httpd created
service/httpd-svc created

root@ubuntu-001:~# kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
httpd-ff8d77b9b-zz4d4   1/1     Running   0          10s

root@ubuntu-001:~# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
httpd-svc    NodePort    10.97.219.48   <none>        8080:30000/TCP   14s
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          69d

进入pod
root@ubuntu-001:~# kubectl exec -it httpd-ff8d77b9b-zz4d4 -- /bin/bash
root@httpd-ff8d77b9b-zz4d4:/usr/local/apache2#

root@httpd-ff8d77b9b-zz4d4:/usr/local/apache2# cat htdocs/index.html
<html><body><h1>It works!</h1></body></html>
```
浏览器输入：http://主机IP:30000/

![1](/images/2020-12-02/1.jpg)


```
root@ubuntu-001:~# cat httpd.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      run: httpd
  template:
    metadata:
      labels:
        run: httpd
    spec:
      containers:
      - name: httpd
        image: httpd
        ports:
        - containerPort: 80
        volumeMounts:
        - name: host-path
          mountPath: /usr/local/apache2/htdocs
      volumes:
      - name: host-path
        hostPath:
           path: /root/test
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  type: NodePort
  selector:
    run: httpd
  ports:
  - protocol: TCP
    nodePort: 30000
    port: 8080
    targetPort: 80

这里进一步使用hostPath将本地test目录，映射到容器的/usr/local/apache2/htdocs目录下

root@ubuntu-001:~# kubectl apply -f httpd.yaml
deployment.apps/httpd configured
service/httpd-svc configured

root@ubuntu-001:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
httpd-589bbcf648-28n9x   1/1     Running   0          7m19s

root@ubuntu-001:~# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
httpd-svc    NodePort    10.104.70.26   <none>        8080:30000/TCP   3h52m
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          69d

```

浏览器输入：http://主机IP:30000/

![2](/images/2020-12-02/2.jpg)


特别说明：这里我使用了hostPath，且httpd副本我修改成了1，实际上，httpd pod可能落在其他node上，所以严格来说，hostPath难以保证所在node上test的存在

可以考虑使用glusterfs做统一的后端存储
