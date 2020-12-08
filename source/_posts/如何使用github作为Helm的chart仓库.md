---
title: 如何使用github作为Helm的chart仓库
date: 2020-12-01 09:35:30
tags:
- GitHub
- helm
categories:
- helm 
---

# 前提条件：
1、已安装git
2、已注册github账户
3、安装helm（推荐helm3，下载地址：[https://github.com/helm/helm/releases/(https://github.com/helm/helm/releases/)）

# 操作步骤：
## 1、在github创建仓库，取名为helm-chart
![创建仓库](https://upload-images.jianshu.io/upload_images/10839544-8d2d6093ca2aff4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 2、下载helm-chart仓库到本地
```
zj@zj-Z390-UD:~/code$ git clone https://github.com/yushanjin/helm-chart.git
正克隆到 'helm-chart'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
展开对象中: 100% (3/3), 完成.
```
## 3、创建chart目录
 ```
zj@zj-Z390-UD:~/code$ cd helm-chart/
zj@zj-Z390-UD:~/code/helm-chart$ ll
总用量 16
drwxr-xr-x 3 zj zj 4096 4月   8 14:04 ./
drwxr-xr-x 4 zj zj 4096 4月   8 14:04 ../
drwxr-xr-x 8 zj zj 4096 4月   8 14:04 .git/
-rw-r--r-- 1 zj zj   77 4月   8 14:04 README.md
zj@zj-Z390-UD:~/code/helm-chart$ helm create test
Creating test

注意：可以在终端执行 source <(helm completion bash)，启动helm命令自动补全功能
zj@zj-Z390-UD:~/code/helm-chart$ tree test/
test/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 9 files
```
## 4、打包chart包
```
zj@zj-Z390-UD:~/code/helm-chart$ helm package test/
Successfully packaged chart and saved it to: /home/zj/code/helm-chart/test-0.1.0.tgz
zj@zj-Z390-UD:~/code/helm-chart$ helm repo index --url https://yushanjin.github.io/helm-chart/ .
zj@zj-Z390-UD:~/code/helm-chart$ cat index.yaml 
apiVersion: v1
entries:
  test:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2020-04-08T14:19:31.079089336+08:00"
    description: A Helm chart for Kubernetes
    digest: 6de7ab4f2da011db9ef8e1def8d2fca7d4e79bb4e81e46152a8d3a2969b73820
    name: test
    type: application
    urls:
    - https://yushanjin.github.io/helm-chart/test-0.1.0.tgz
    version: 0.1.0
generated: "2020-04-08T14:19:31.078672885+08:00"
```
## 5、push到github
```
zj@zj-Z390-UD:~/code/helm-chart$ git status
位于分支 master
您的分支与上游分支 'origin/master' 一致。

未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）

	index.yaml
	test-0.1.0.tgz
	test/

提交为空，但是存在尚未跟踪的文件（使用 "git add" 建立跟踪）
zj@zj-Z390-UD:~/code/helm-chart$ git add .
zj@zj-Z390-UD:~/code/helm-chart$ git status
位于分支 master
您的分支与上游分支 'origin/master' 一致。

要提交的变更：
  （使用 "git reset HEAD <文件>..." 以取消暂存）

	新文件：   index.yaml
	新文件：   test-0.1.0.tgz
	新文件：   test/.helmignore
	新文件：   test/Chart.yaml
	新文件：   test/templates/NOTES.txt
	新文件：   test/templates/_helpers.tpl
	新文件：   test/templates/deployment.yaml
	新文件：   test/templates/ingress.yaml
	新文件：   test/templates/service.yaml
	新文件：   test/templates/serviceaccount.yaml
	新文件：   test/templates/tests/test-connection.yaml
	新文件：   test/values.yaml

zj@zj-Z390-UD:~/code/helm-chart$ git commit -m "创建test的chart包"
[master 5ae813c] 创建test的chart包
 12 files changed, 348 insertions(+)
 create mode 100644 index.yaml
 create mode 100644 test-0.1.0.tgz
 create mode 100644 test/.helmignore
 create mode 100644 test/Chart.yaml
 create mode 100644 test/templates/NOTES.txt
 create mode 100644 test/templates/_helpers.tpl
 create mode 100644 test/templates/deployment.yaml
 create mode 100644 test/templates/ingress.yaml
 create mode 100644 test/templates/service.yaml
 create mode 100644 test/templates/serviceaccount.yaml
 create mode 100644 test/templates/tests/test-connection.yaml
 create mode 100644 test/values.yaml
zj@zj-Z390-UD:~/code/helm-chart$ git push origin master
Username for 'https://github.com': yushanjin
Password for 'https://yushanjin@github.com': 
对象计数中: 17, 完成.
Delta compression using up to 8 threads.
压缩对象中: 100% (16/16), 完成.
写入对象中: 100% (17/17), 8.32 KiB | 1.66 MiB/s, 完成.
Total 17 (delta 0), reused 0 (delta 0)
To https://github.com/yushanjin/helm-chart.git
   48e2023..5ae813c  master -> master
```
注意： 我这里本地没有创建其他分支，所以直接push到master分支了
## 6、设置github上helm-chart仓库的GitHub Pages（在仓库的settings里设置）
![image.png](https://upload-images.jianshu.io/upload_images/10839544-fece4f06795a48d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
登录：https://yushanjin.github.io/helm-chart，页面内容为README.md的内容
![image.png](https://upload-images.jianshu.io/upload_images/10839544-9f84217a461cce46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**注意**： 我修改过README.md文件
## 7、本地添加自己的chart仓库
```
zj@zj-Z390-UD:~/code/helm-chart$ helm repo list
Error: no repositories to show
zj@zj-Z390-UD:~/code/helm-chart$ helm repo add myrepo https://yushanjin.github.io/helm-chart
"myrepo" has been added to your repositories
zj@zj-Z390-UD:~/code/helm-chart$ helm repo list
NAME  	URL                                   
myrepo	https://yushanjin.github.io/helm-chart
zj@zj-Z390-UD:~/code/helm-chart$ helm search repo test
NAME       	CHART VERSION	APP VERSION	DESCRIPTION                
myrepo/test	0.1.0        	1.16.0     	A Helm chart for Kubernetes
```

至此，就可以使用**helm install xxx myrepo/test** 安装test了
