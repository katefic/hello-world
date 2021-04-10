## 1.kafka架构

包含

​	Producer。

​	Broker：具体的服务器

​	Consumer(Group)、

## 2.topics和partitions

在kafka的配置文件$KAFKA_HOME/config/server.properties中有记录日志删除策略；对于consumer来pull消息不需要锁机制

**kafka默认使用端口9092**

```shell
broker.id=1
delete.topic.enable=true
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
controlled.shutdown.enable=true
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
```

