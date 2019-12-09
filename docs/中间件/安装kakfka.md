## 安装kakfka
从官网下载Kafka安装包，解压安装，或直接使用命令下载。

wget [**http://mirror.bit.edu.cn/apache/kafka/2.1.1/kafka_2.11-2.1.1.tgz**](http://mirror.bit.edu.cn/apache/kafka/2.1.1/kafka_2.11-2.1.1.tgz) 

解压安装
>tar -zvxf kafka_2.11-2.1.1.tgz -C /usr/local/
cd /usr/local/kafka_2.11-2.1.1/

修改配置文件
>vim config/server.properties 

修改其中
>broker.id=1
log.dirs=data/kafka-logs

## 功能验证
#### 启动zookeeper
使用安装包中的脚本启动单节点Zookeeper实例：
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

#### 启动Kafka服务
使用kafka-server-start.sh启动kafka服务：
bin/kafka-server-start.sh config/server.properties
进程守护模式启动kafka
nohup bin/kafka-server-start.sh config/server.properties >/dev/null 2>&1 & 

#### Kafka关闭命令(备注：先进入kafka目录)
bin/kafka-server-stop.sh

#### 创建Topic
使用kafka-topics.sh 创建但分区单副本的topic test
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

#### 查看Topic
bin/kafka-topics.sh --list --zookeeper localhost:2181

#### 产生消息
使用kafka-console-producer.sh 发送消息
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test 

#### 消费消息
使用kafka-console-consumer.sh 接收消息并在终端打印
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

#### 删除Topic
bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic test

#### 查看描述 Topic 信息
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
第一行给出了所有分区的摘要，每个附加行给出了关于一个分区的信息。 由于我们只有一个分区，所以只有一行。
“Leader”: 是负责给定分区的所有读取和写入的节点。 每个节点将成为分区随机选择部分的领导者。
“Replicas”: 是复制此分区日志的节点列表，无论它们是否是领导者，或者即使他们当前处于活动状态。
“Isr”: 是一组“同步”副本。这是复制品列表的子集，当前活着并被引导到领导者。

## 集群配置
Kafka支持两种模式的集群搭建：

单机多broker集群配置；
多机多broker集群配置。

#### 单机多breoker
利用单节点部署多个broker。不同的broker不同的id，监听端口以及日志目录，如：

将配置文件复制两份
cp config/server.properties config/server-1.properties
cp config/server.properties config/server-2.properties 
1
2
修改配置文件信息
vim config/server-1.properties
#修改内容
broker.id=2
listeners=PLAINTEXT://your.host.name:9093
log.dirs=/data/kafka-logs-1


vim config/server-2.properties
#修改内容
broker.id=3
listeners=PLAINTEXT://your.host.name:9094
log.dirs=/data/kafka-logs-2
1
2
3
4
5
6
7
8
9
10
11
12
启动多个kafka服务
in/kafka-server-start.sh config/server-1.properties 

bin/kafka-server-start.sh config/server-2.properties 
1
2
3
最后按照上面方法产生和消费信息。

#### 多机多broker
分别在多个节点按上述方式安装Kafka，配置启动多个Zookeeper 实例。如：192.168.18.130、192.168.18.131、192.168.18.132三台机器

分别配置多个机器上的Kafka服务 设置不同的broke id，zookeeper.connect设置如下:
zookeeper.connect=192.168.18.130:2181,192.168.18.131:2181,192.168.18.132:2181
--------------------- 




bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties

bin /kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning

mysql: bin/connect-standalone.sh config/connect-standalone.properties /data/kafka-config/quickstart-mysql.properties  /data/kafka-config/quickstart-mysql-sink.properties 



bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic mysql-kafka-comments



bin/connect-standalone.sh config/connect-standalone.properties /data/kafka-config/timestamp-mysql-source.properties /data/kafka-config/timestamp-mysql-sink.properties 

作者：wqh3520 
来源：CSDN 
原文：https://blog.csdn.net/wqh8522/article/details/79163467 
版权声明：本文为博主原创文章，转载请附上博文链接！





http://kafka.apache.org/quickstart