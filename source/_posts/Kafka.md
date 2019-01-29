---
title:  Install kafka on Win10 with WSL
data: 2018-10-28
---
# Kafka

## Install kafka on Win10 WSL

### Install Java

- `sudo apt update`

- `sudo apt install openjdk-8-jdk`

  `/usr/lib/jvm`

### Install zookeeper

- `sudo apt install zookeeper`

  `/usr/share/zookeeper`

  `/etc/zookeeper`

- `sudo /usr/share/zookeeper/bin/zkServer.sh start`

- `sudo /usr/share/zookeeper/bin/zkServer.sh status`

- `telnet localhost 2181`

### Install Kafka

- 下载

  `wget http://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.0.0/kafka_2.11-2.0.0.tgz`

- 解压&安装

  `sudo tar -zxf kafka_2.11-2.0.0.tgz`

  `sudo mkdir -p /usr/local/kafka`

  `sudo mv kafka_2.11-2.0.0 /usr/local/kafka`

  `sudo mkdir -p /tmp/kafka-logs`

- 启动Kafka

  `sudo cd /usr/local/kafka/kafka_2.11-2.0.0`

  `sudo bin/kafka-server-start.sh -daemon config/server.properties`

- 测试安装

  - 创建主题

    `sudo bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test`

    `sudo bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test`

    `sudo bin/kafka-topics.sh --list --zookeeper localhost:2181`

  - 发布消息

    `sudo bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test`

  - 读取消息

    `sudo bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning`