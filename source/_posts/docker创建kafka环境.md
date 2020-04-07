---
title: docker创建kafka环境
tags:
  - null
categories:
  - null
date: 2020-04-06 16:52:20
---

1. 下载镜像

```shell
docker pull zookeeper:3.5.6
```

```shell
docker pull wurstmeister/kafka
```

2. 启动

```shell
docker run -d --name zookeeper -p 2181:2181 -t zookeeper:3.5.6
```

```shell
docker run -d --name kafka --publish 9092:9092 --link zookeeper --env KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 --env KAFKA_ADVERTISED_HOST_NAME=localhost --env KAFKA_ADVERTISED_PORT=9092  wurstmeister/kafka:latest
```

可通过`docker ps`查看启动状态

3. 测试

`docker ps`找到container id，进入容器

```shell
# docker exec -it ${CONTAINER NAME} /bin/bash
docker exec -it kafka /bin/bash
```

进入kafka默认目录

```shell
cd /opt/kafka_2.12-2.4.1/bin
```



4. 创建主题

```shell
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
```

```shell
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```



5. 发送消息

```shell
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```



6. 消费消息

```shell
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```





7. 多个kafka节点

``` shell
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 1 --topic my-replicated-topic1
```



```shell
docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=192.168.199.246:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.199.246:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka
```

```shell
docker run -d --name kafka1 -p 9093:9093 -e KAFKA_BROKER_ID=1 -e KAFKA_ZOOKEEPER_CONNECT=192.168.199.246:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.199.246:9093 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9093 -t wurstmeister/kafka
```

```shell
docker run -d --name kafka2 -p 9094:9094 -e KAFKA_BROKER_ID=2 -e KAFKA_ZOOKEEPER_CONNECT=192.168.199.246:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.199.246:9094 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9094 -t wurstmeister/kafka
```





### 删除主题

```shell
bin/kafka-topics.sh --delete --topic my-replicated-topic --bootstrap-server localhost:9092
```



查看主题

```shell
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
```

```shell
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test1
```

```shell
bin/kafka-topics.sh --describe --zookeeper 192.168.199.246:2181 --topic test1
```





参考文章

[在Docker环境下部署Kafka](https://blog.csdn.net/qq_35394891/article/details/84349955)

[Mac 使用 docker 搭建 kafka 集群 + Zookeeper + kafka-manager](https://www.jianshu.com/p/fe73765ef74d)

[apache kafka quickstart](https://kafka.apache.org/documentation/#quickstart)

