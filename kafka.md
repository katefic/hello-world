## 1.kafka架构

包含

​	Producer。

​	Broker：具体的服务器

​	Consumer(Group)、

Kafka使用ZooKeeper对集群元数据进行持久化存储，如果ZooKeeper丢失了Kafka数据，集群的副本映射关系以及topic等配置信息都会丢失，最终导致Kafka集群不再正常工作，造成数据丢失的后果。

## 2.topics和partitions

在kafka的配置文件$KAFKA_HOME/config/server.properties中有记录日志删除策略；对于consumer来pull消息不需要锁机制

**kafka默认使用端口9092**

```shell
#这三项都配置
broker.id=1
delete.topic.enable=true
或者用listener
host.name=172.16.84.21


num.network.threads=16
num.io.threads=48
background.threads=24
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
min.insync.replicas=2
unclean.leader.election.enable=false
group.max.session.timeout.ms=3600000

auto.create.topics.enable=true

queued.max.requests=5000
connections.max.idle.ms=600000

#It will migrate any partitions the server is the leader for to other replicas prior to shutting down
controlled.shutdown.enable=true

#ISR踢出策略，默认10000ms
replica.lag.time.max.ms = 10000
controlled.shutdown.max.retries=3
controlled.shutdown.retry.backoff.ms=5000
controller.socket.timeout.ms=30000
message.max.bytes=2000000
replica.fetch.max.bytes=2500000
auto.leader.rebalance.enable=true
leader.imbalance.check.interval.seconds=120
num.replica.fetchers=8
log.roll.hours=168
log.segment.delete.delay.ms=60000
###########消息存放的目录，这个目录可以配置为“，”逗号分割的表达式
log.dirs=/data/kafka/kafka-logs
##########新建Topic的默认Partition数量#################
num.partitions=1
default.replication.factor=3
log.cleaner.threads=8
num.recovery.threads.per.data.dir=8
##############日志删除策略##########################
log.retention.hours=360
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
###############Kafka通过Zookeeper管理集群配置，选举leader########################
zookeeper.connect=172.16.84.21:2181,172.16.84.22:2181,172.16.84.23:2181
zookeeper.connection.timeout.ms=6000
zookeeper.session.timeout.ms=6000
zookeeper.sync.time.ms=2000
offsets.retention.minutes=10080
group.initial.rebalance.delay.ms=3
max.in.flight.requests.per.connection=5

```

## 3.Producer消息路由

不同的消息可以并行写入不同broker的不同Partition里，极大的提高了吞吐率

## 4.启动

```shell
#先启动zookeeper集群
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.4.11
bin/zkServer.sh start [conf/zoo0.cfg]
#启动命令
export KAFKA_HOME=/usr/local/kafka_2.12-1.0.0
bin/kafka-server-start.sh config/server.properties

zkCli.sh
get /brokers/topics/__consumer_offsets/partitions/40/state
```

## 5.常用命令

此时消费者可接收到消息，但是顺序和生产者的发送顺序不一样，也就是说当一个主题有多个分区时，消费者接受消息的顺序是乱序的，因为每个partition的网络性能不一样，读写性能也不一样。

重置消费组offsets时，该组应该不是正在消费

Assignments can only be reset if the group 'console-consumer-16754' is inactive

```shell
##列出消费组
kafka-consumer-groups.sh --bootstrap-server 2.2.11.79:9092 --lists
#重置offsets
kafka-consumer-groups.sh --bootstrap-server 2.2.11.79:9092 --group console-consumer-16754 --topic s2 --execute --reset-offsets --to-earliest
#模拟消费
kafka-console-consumer.sh --bootstrap-server 2.2.11.79:9092 --group console-consumer-16754 --topic s2
#当提示group选项不可用时
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic s2 --consumer-property group.id=1 --from-beginning
#或者
--consumer.config config.file
```

0.8以前的kafka，消费的进度(offset)是写在zk中的，所以consumer需要知道zk的地址。后来的版本都统一由broker管理，所以就用bootstrap-server了。



连接多台只是保证其中一台挂了，producer还能连接上kafka集群，producer生产的消息还是会以消息指定的分区或者key或者轮询或者随机的存在kafka集群的某个节点上



**zk的leader和kafka的controller不是同一个**

## 6.问题

```shell
kafka-consumer-groups.sh --bootstrap-server 2.2.11.79:9092 --describe --group g1
```



Consumer group 'g1' is rebalancing

可能是参数问题，使用em的配置后，再使用生产、消费、重置命令是正常的
2021-04-18
在笔记本中用单台虚拟机搭建zk和kafka集群，也没有办公机上的3台卡

## 7.

要启用幂等性，只需将 Producer的参数中 的参数中 的参数中 enable.idompotence设置为 设置为 true即可



