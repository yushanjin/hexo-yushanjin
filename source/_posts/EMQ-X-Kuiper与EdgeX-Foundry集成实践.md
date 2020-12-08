---
title: EMQ X Kuiper与EdgeX Foundry集成实践
date: 2020-12-01 10:34:44
tags:
- kuiper
- edgex
categories:
- kuiper
- edgex
---

Kuiper是什么? EdgeX Foundry又是什么？

# Kuiper
EMQ X Kuiper 是 Golang 实现的轻量级物联网边缘分析、流式处理开源软件，可以运行在各类**资源受限的边缘设备**上。Kuiper 设计的一个主要目标就是将在云端运行的实时流式计算框架（比如 [Apache Spark](https://spark.apache.org/)，[Apache Storm](https://storm.apache.org/) 和 [Apache Flink](https://flink.apache.org/) 等）迁移到边缘端。Kuiper 参考了上述云端流式处理项目的架构与实现，结合边缘流式数据处理的特点，采用了编写**基于`源 (Source)`，`SQL (业务逻辑处理)`, `目标 (Sink)` 的规则引擎来实现边缘端的流式数据处理**。
其架构如下：
![image.png](https://upload-images.jianshu.io/upload_images/10839544-82b09a3d6c9e5c33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 源 (Sources) ：内置支持 MQTT 数据的接入，扩展支持与EdgeX Foundry集成
- SQL：流式数据逻辑处理，具备完整的数据分析处理能力
   - 支持丰富的数据类型
   - 支持4种时间窗口（滚动窗口、跳跃窗口、滑动窗口、会话窗口）
   - 内置60+处理函数
   - 提供类SQL语句对数据进行抽取、过滤、转换
- 目标(Sinks)：内置支持 MQTT、HTTP等

EMQ公司的相关产品，可以登陆其官网查询了解
![image.png](https://upload-images.jianshu.io/upload_images/10839544-b1643f3d02bea968.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# EdgeX Foundry
EdgeX Foundry是一个Linux 基金会运营的开源的，基于与硬件和操作系统完全无关的边缘计算物联网软件框架项目。其是一系列松耦合、开源的微服务集合，位于网络的边缘，可以与设备、传感器、执行器和其他物联网对象的物理世界进行交互。EdgeX Foundry 旨在创造一个互操作性、即插即用、模块化的物联网边缘计算的生态系统。
其架构如下：
![image.png](https://upload-images.jianshu.io/upload_images/10839544-fd06dab15f5598af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从架构图可以看出：
**南侧（SouthBound）**:在物理领域内的所有物联网对象，以及与这些设备、传感器、执行器和其他物联网对象直接通信并从中收集数据的网络边缘，统称为“南侧”。
**北侧（NorthBound）**:将数据收集、存储、聚合、分析并转换为信息的云(或企业系统)，以及与云通信的网络部分称为网络的“北侧”。
因此，EdgeX使数据可以向北移动到云，也可以横向移动到其他网关，或返回到设备、传感器和执行器。
EdgeX的重要服务层及微服务：
![image.png](https://upload-images.jianshu.io/upload_images/10839544-d0e4c5a219ab700d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 1、安装edgex
参照官网文档：[https://fuji-docs.edgexfoundry.org/Ch-GettingStartedUsers.html](https://fuji-docs.edgexfoundry.org/Ch-GettingStartedUsers.html)
docker-compose启动
![image.png](https://upload-images.jianshu.io/upload_images/10839544-f5a8d9ed341b66b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
相关服务正常
![image.png](https://upload-images.jianshu.io/upload_images/10839544-8ab54653eca76cc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2、安装并启动kuiper
sudo docker run -d --name kuiper --restart always -e EDGEX_SERVER=10.0.105.143 -e EDGEX_PORT=5563 -e EDGEX_SERVICE_SERVER=http://10.0.105.143:48080 emqx/kuiper:0.2.1

环境变量具体参考：[https://hub.docker.com/r/emqx/kuiper](https://hub.docker.com/r/emqx/kuiper) 中的说明
EDGEX_SERVER：edgex中zeromq的地址（zeromq集成到core data服务中了，可以看到core data服务暴露了两个端口一个5563，一个48080）
EDGEX_PORT：edgex中zeromq的端口
EDGEX_SERVICE_SERVER：edgex中core data的地址及端口
这里我使用的kuiper镜像为 emqx/kuiper:0.2.1，为目前最新版本

## 3、进入kuiper容器
sudo docker exec -it kuiper /bin/sh

## 4、查看日志
/kuiper # cat log/stream.log

## 5、创建流，订阅来自edgex的消息流
/kuiper # bin/cli create stream demo'() WITH (FORMAT="JSON", TYPE="edgex")'

## 6、创建规则文件，内容如下
/kuiper # cat rule.txt
```
{
  "sql": "SELECT * from demo GROUP BY TUMBLINGWINDOW(ss, 10)",
  "actions": [
    {
      "mqtt": {
        "server": "tcp://broker.emqx.io:1883",
        "topic": "result",
        "clientId": "demo_001"
      }
}
  ]
}
```
上述规则：Kuiper将接受edgex的数据，执行select操作（每10s钟），然后将处理后的数据发布到tcp://broker.emqx.io:1883（也可以换成其他的，比如broker.hivemq.com或者自己搭建的EMQ X edge）
**注意：**Kuiper SQL相关的参考，见https://docs.emqx.io/kuiper/latest/cn/sqls/overview.html

## 7、创建规则，命名为rule1
/kuiper # bin/cli create rule rule1 -f rule.txt

## 8、查看日志，可以看到已经连通edgex，相关的规则也已经创建
![image.png](https://upload-images.jianshu.io/upload_images/10839544-4f07b16e3735e2dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 9、查看规则状态
![image.png](https://upload-images.jianshu.io/upload_images/10839544-1349f6fd6210e627.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 10、使用mosquitto订阅broker.emqx.io中主题为result的消息
![image.png](https://upload-images.jianshu.io/upload_images/10839544-4fdbca2eb0e7af64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


分析结果，发布到文件（[https://github.com/emqx/kuiper/blob/master/docs/zh_CN/plugins/sinks/file.md](https://github.com/emqx/kuiper/blob/master/docs/zh_CN/plugins/sinks/file.md)）
```
{
  "sql": "SELECT * from demo",
  "actions": [
   {
      "file": {
        "path": "/tmp/result.txt",
        "interval": 5000
      }
   }
  ]
}
```
分析结果，发布到zmq（[https://github.com/emqx/kuiper/blob/master/docs/zh_CN/plugins/sinks/zmq.md](https://github.com/emqx/kuiper/blob/master/docs/zh_CN/plugins/sinks/zmq.md)）
```
{
  "sql": "SELECT * from demo",
  "actions": [
  {
    "zmq": {
       "server": "tcp://127.0.0.1:5563",
       "topic": "temp"
      }
   }
  ]
}
```
分析结果，通过调用rest（[https://github.com/emqx/kuiper/blob/master/docs/en_US/rules/sinks/rest.md](https://github.com/emqx/kuiper/blob/master/docs/en_US/rules/sinks/rest.md)）
```
{
  "sql": "SELECT * from demo",
  "actions": [
  {
    "rest": {
      "url": "http://127.0.0.1:48082/api/v1/device/cc622d99-f835-4e94-b5cb-b1eff8699dc4/command/51fce08a-ae19-4bce-b431-b9f363bba705",       
      "method": "post",
      "dataTemplate": "\"newKey\":\"{{.key}}\"",
      "sendSingle": true
      }
    }
  ]
}
```
**这个类似于EdgeX中core services中的command服务**

分析结果，发布到edgex（[https://github.com/emqx/kuiper/blob/master/docs/en_US/rules/sinks/edgex.md](https://github.com/emqx/kuiper/blob/master/docs/en_US/rules/sinks/edgex.md)）
```
{
  "sql": "SELECT * from demo",
  "actions": [
    {
      "edgex": {
        "protocol": "tcp",
        "host": "*",
        "port": 5571,
        "topic": "application",
        "deviceName": "kuiper",
        "contentType": "application/json"
      }
    }
  ]
}
```
分析结果，发布到日志文件，默认在log/stream.log
```
{
  "sql": "SELECT * from demo",
  "actions": [
    {
      "log": {}
    }
  ]
}
```
经验证，有些插件不完整

```
/kuiper # ./bin/cli getstatus rule rule1
Connecting to 127.0.0.1:20498... 
Stopped: cannot open /kuiper/plugins/sinks/File.so: plugin.Open("/kuiper/plugins/sinks/File.so"): Error relocating /kuiper/plugins/sinks/File.so: __fprintf_chk: symbol not found.
```


参考：

1、kuiper官方文档及github地址

[https://docs.emqx.io/kuiper/latest/cn/](https://docs.emqx.io/kuiper/latest/cn/)

[https://github.com/emqx/kuiper](https://github.com/emqx/kuiper)

2、kuiper集成edgex文档[https://github.com/emqx/kuiper/blob/master/docs/en_US/edgex/edgex_rule_engine_tutorial.md](https://github.com/emqx/kuiper/blob/master/docs/en_US/edgex/edgex_rule_engine_tutorial.md)
