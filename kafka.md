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
#在可用性和消息丢失之间做出权衡
unclean.leader.election.enable=false


group.max.session.timeout.ms=3600000

auto.create.topics.enable=true

queued.max.requests=5000
connections.max.idle.ms=600000

#It will migrate any partitions the server is the leader for to other replicas prior to shutting down
controlled.shutdown.enable=true

#ISR踢出策略，默认10000ms，参数在官文中可查
#消费者只能看到已经复制到ISR的消息
replica.lag.time.max.ms = 10000

controlled.shutdown.max.retries=3
controlled.shutdown.retry.backoff.ms=5000
controller.socket.timeout.ms=30000

#单个消息被压缩后的最大值，默认1M（1000 000）
#这个值对性能有显著的影响
#消费者客户端设置fetch.max.bytes的必须与服务器端设置的消息
#大小进行协调
message.max.bytes=2000000
fetch.max.bytes=
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
#每个主题可以设置单独的保留策略
log.retention.hours=360
log.retention.bytes=默认1G

##################################################
acks=all
buffer.memory=
compression.type=
retries=
batch.size=
client.id=
linger.ms

#元数据刷新时间
metadata.max.age.ms

#日志片断（segment）被关闭后，才会计算上面的过期信息
log.segment.bytes=1073741824
log.segment.ms=
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

#查看kafka版本
ll libs/kafka*.jar
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
#################################
#这些脚本都是调用kafka-run-class.sh，以一个对应的类名为参数，来启动一个java进程
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

## 8. 生产者分区

按指定partition，有key value ，无key来分别存储，数据实际存储的位置是在*logs.dir*下的*topic-partition*目录下的.log文件中，按轮循的方式存储到broker的partition中

分区数的选择：

如果你估算出主题的吞吐量和悄费者吞吐量，可以**用主题吞吐量除以消费者吞吐量算出分区的个数**。也就是说，
如果每秒钟要从主题上写入和读取lGB 的数据，并且每个消费者每秒钟可以处理50MB的数据，那么至少需要20 个分区。这样就可以让20 个消费者同时读取这些分区，从而达到每秒钟lGB 的吞吐量。

如果不知道这些信息，那么根据经验，把分区的大小限制在25GB 以内可以得到比较理想
的效果。

**在日志片段被关闭之前消息是不会过期的**

磁盘性能影响生产者，而内存影响消费者。不建议把Kafka 同其他重要的应用程序部署
在一起的原因，它们需要共享页面缓存，最终会降低Kafka 消费者的性能。



使用集群最大的好处是可以跨服务器进行负载均衡，再则就是可以使用复制功能来避免因单点故
障造成的数据丢失

## 9.操作系统调优

### 9.1 虚拟内存

​	#建议把vm.swappiness 参数的值设置得小一点，比如1

​	/proc/sys/vm/swappiness

​	/proc/sys/vm/dirty_background_ratio 	//小于10

​	/proc/sys/vm/dirty_ratio 	//60-80比较合理

```shell
#为了给这些参数设置合适的值，最好是在Kafka 集群运行期间检查脏页的数量
grep -E 'dirty|writeback' /proc/vmstat
```

### 9.2 磁盘

​        文件元数据包含3 个时间戳： 创建时间（ ctime ）、最后修改时间（ mtime）以及最后访问时间（ atime）。默认情况下，每次文件被读取后都会更新atime ，这会导致大量的磁盘写操作。Kafka 用不到该属性，所以完全可以把它禁用掉。为挂载点设置noatime 参数可以防止更新atime

```shell
修改/etc/fstab，如下
/dev/sdb1 /home/disk0 ext4 defaults 0 2
改成
/dev/sdb1 /home/disk0 ext4 noatime 0 2
修改/etc/fstab设置后需要重新挂载文件系统、不必重启就可以应用新设置。remount分区，执行：
mount -o remount /home/disk0

如果不想改fstab，直接用mount命令：
mount -o noatime,remount /home/disk0
```



#### 9.3 网络

```shell
#socket读写缓冲区，单位byte
net.core.wmem_default
net.core.rmem_default
net.core.wmem_max=2097152
net.core.rmem_max=2097152

#tcp socket的读写缓冲区，这些参数的值由3 个整数组成，它们使用空格分隔，分别
#表示最小值、默认值和最大值
#例如，“4096 65536 2048000”表示最小值是4K、默认值是64K、最大值
#是2MB 。根据Kafka 服务器接收流量的实际情况，可能需要设置更高的最大值
net.ipv4.tcp_wmem
net.ipv4.tcp_rmem

#启用tcp时间窗扩展
net.ipv4.tcp_window_scaling=1

#可以接收更多的并发连接
net.ipv4.tcp_max_syn_backlog		//大于1024

#大于1000，有助于应对网络流量的爆发，特别是千兆网络
net.core.netdev_max_backlog
```

## 10.消费者

在Kafka 和其他系统之间复制数据时，使用正则表达式的方式订阅多个主题是很常见的做法

消费者往一个叫作_consumer_offset 的特殊主题发送消息，消息里包含每个分区的偏移量

## 11.

集群里第一个启动的broker 通过在Zookeeper 里创建一个临时节点/controller让自己成为控制器，控制器负责在节点加入或离开集群时进行分区首领选举

Kafka 最为常见的几种请求类型：元数据请求、生产请求和获取请求



主题级别的配置参数是replication.factor，而在broker 级别则可以通过default.replication.factor来配置自动创建的主题。



建议把broker 分布在多个不同的机架上，并使用broker.rack 参数来为每个broker 配置所在机架的名字。如果配置了机架名字， Kafka 会保证分区的副本被分布在多个机架上，从而获得更高的可用性

enable.auto.commit

auto.commit.interval.ms

## 12. connector

```shell
echo '{ "name": "load-kafka-config", "config": { "connector.class": "FileStreamSourceConnector", "file": "config/server.properties", "topic": "kafka-config-topic" } }' | curl -XPOST -d@- http://localhost:8083/connectors --header "content-type:application/json"

#执行这个没有生成新文件
curl -s -X POST -H "Content-Type: application/json" --data '{"name": "dump-kafka-config", 
   "config":
    {"connector.class":"org.apache.kafka.connect.file.FileStreamSinkConnector",
    "key.converter.schemas.enable":"true",
    "file":"demo-file.txt",
    "tasks.max":"1",
    "value.converter.schemas.enable":"true",
    "name":"file-stream-demo-distributed",
    "topics":"kafka-config-topic",
    "value.converter":"org.apache.kafka.connect.json.JsonConverter",
    "key.converter":"org.apache.kafka.connect.json.JsonConverter"}
 }'  http://localhost:8083/connectors/

```

