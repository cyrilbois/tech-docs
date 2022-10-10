---
title:  Kafka 快速开始
description: Kafka 快速安装运行调试及Spring Boot 集成
...

主要用于测试环境的安装、启动、测试及Spring Boot的集成

### 下载
从官网下载一个Kafka稳定版本，这里采用的是Kafka 2.11-1.1.1版本 http://kafka.apache.org/downloads

官方安装指南： [官方-Quick Start](https://kafka.apache.org/documentation/#quickstart)

```
wget https://archive.apache.org/dist/kafka/1.1.1/kafka_2.11-1.1.1.tgz
tar -xzf kafka_2.13-3.3.1.tgz
```

### 启动
#### Start the ZooKeeper service
```
./bin/zookeeper-server-start.sh config/zookeeper.properties
```
####  Start the Kafka broker service
```
bin/kafka-server-start.sh config/server.properties
```
> 后台运行 `bin/kafka-server-start.sh config/server.properties &`


### 停止
执行脚本 `./bin/kafka-server-stop.sh`

### 配置

在config/server.properties文件中
```
#listeners=PLAINTEXT://:9092 
listeners=PLAINTEXT://10.3.69.41:9092 # 测试环境，修改这个基本就可以了
```


### Topic 操作
#### 创建Topic
```
bin/kafka-topics.sh --create --zookeeper 127.0.0.1:2181 --replication-factor 1 --partitions 1 --topic test
```
#### 列出所有Topic
```
bin/kafka-topics.sh --list --zookeeper 127.0.0.1:2181
```
#### 删除Topic
```
bin/kafka-topics.sh --delete --zookeeper 127.0.0.1:2181 --topic test
#如果没有在配置里设置彻底删除Topic，此处则只是将该Topic标志为删除
```

### 测试发送数据

```
bin/kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic test
// 输入内容
> {"id": "1001","msg":"Hello"}
> 文本消息
```
 

### 测试消费数据
```
bin/kafka-console-consumer.sh --zookeeper 127.0.0.1:2181 --topic test --from-beginning
// 输出内容
{"id": "1001","msg":"Hello"}
文本消息
```