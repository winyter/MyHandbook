# 零碎的 Kafka 知识

修改 Topic 分区数

```shell
kafka-topics.sh --alter --topic one14 --zookeeper rhino74:2181 --partitions 3
```

查看消费组消费情况

```shell
kafka-consumer-groups.sh --bootstrap-server rhino221:9092 --group zww_topic_group510 --describe
```

