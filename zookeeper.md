## 1.简介

ZooKeeper是一种为分布式应用所设计的高可用、高性能且一致的开源协调服务，它提供了一项基本服务：**分布式锁服务**

## 2.安装

集群模式搭建

/usr/local/zookeeper-3.4.11/conf/zoo.cfg

```shell
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
#伪集群时目录不能相同，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里
dataDir=/data/zookeeper/data
#应该谨慎的选择日志存放的位置，使用专用的日志存储设备能够大大提高系统的性能，如果将日志存储在比较繁忙的存储设备上，那么将会很大程度上影像系统性能
dataLogDir=/data/zookeeper/log
# the port at which the clients will connect
#伪集群时注意端口不能相同
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#伪集群时注意端口不能相同
#(3) server.A=B：C：D，C是Zookeeper服务器之间的通信端口，D是选举端口
server.1=172.16.84.21:2287:3387
server.2=172.16.84.22:2287:3387
server.3=172.16.84.23:2287:3387

#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
  utopurge.purgeInterval=1
~                                    
```

$dataDir目录下myid和server.***id***要一致

bin/zkServer.sh为启动脚本

```shell
#启动
bin/zkServer.sh start [conf/zoo0.cfg]
查看状态
bin/zkServer.sh status conf/zoo1.cfg

echo ruok|nc localhost 2181
echo conf|nc localhost 2181

#启动Zookeeper服务之后，输入以下命令，连接到Zookeeper服务：
bin/zkCli.sh -server localhost:2181
```



