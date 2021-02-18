---
title: Kafka学习笔记
author: huimingz
category: [facility]
tags: [facility, kafka]
pin: false
---

环境: MacOS 10.15

## 安装
简单至上，直接使用Homebrew进行安装：
```shell
$ brew install kafka
```

安装完成后会出现下面的提示信息：
```
==> zookeeper
To have launchd start zookeeper now and restart at login:
  brew services start zookeeper
Or, if you don't want/need a background service you can just run:
  zkServer start
==> kafka
To have launchd start kafka now and restart at login:
  brew services start kafka
Or, if you don't want/need a background service you can just run:
  zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties & kafka-server-start /usr/local/etc/kafka/server.properties
```

这段信息说明了如何启用kafka。kafka是基于zookeeper的，所以在启动kafka之前，我们还需要先启动zookeeper。
:w
## 启动
需要先启动zookeeper，在启动kafka，注意顺序。

```shell
# 启动zookeeper
$ brew services start zookeeper

# 启动kafka
brew services start kafka
```

输出:
```
$ brew services start zookeeper
==> Successfully started `zookeeper` (label: homebrew.mxcl.zookeeper)

$ brew services start kafka
==> Successfully started `kafka` (label: homebrew.mxcl.kafka)
```

## 关闭
关闭顺序和启动顺序相反，需要先关闭kafka，再关闭zookeeper。
```
$ brew services stop kafka
$ brew services stop zookeeper
```

## 使用

### 单个代理
1. 创建一个topic:
    ```shell
    $ kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
    Created topic test.
    ```
    使用list命令来查看这个topic:
    ```shell
    $ kafka-topics --list --zookeeper localhost:2181
    __consumer_offsets
    test
    ```
2. 发送消息: 运行producer，然后在控制台输入一些消息以发送到服务器。
    ```shell
    $ kafka-console-producer --broker-list localhost:9092 --topic test
    >This is a message
    >This is another message
    ```
3. 消费消息：启动一个终端consumer进行消息消费。
    ```shell
    $ kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning
    This is a message
    This is another message
    ```

## 参考
1. [kafka官网](https://kafka.apache.org/)
2. [Apache Kafka Installation on Mac using Homebrew](https://medium.com/@Ankitthakur/apache-kafka-installation-on-mac-using-homebrew-a367cdefd273)
3. [homebrew formula](https://formulae.brew.sh/formula/kafka)

