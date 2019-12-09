介绍
Kafka是分布式发布-订阅消息系统，最初由LinkedIn公司开发，之后成为之后成为Apache基金会的一部分，由Scala和Java编写。Kafka是一种快速、可扩展的、设计内在就是分布式的，分区的和可复制的提交日志服务。

它与传统系统相比，有以下不同：

它被设计为一个分布式系统，易于向外扩展；
它同时为发布和订阅提供高吞吐量；
它支持多订阅者，当失败时能自动平衡消费者；
它将消息持久化到磁盘，因此可用于批量消费，例如ETL，以及实时应用程序。
基础概念
Broker：Kafka集群包含一个或多个服务器，这些服务器就是Broker
Topic：每条发布到Kafka集群的消息都必须有一个Topic
Partition：是物理概念上的分区，为了提供系统吞吐率，在物理上每个Topic会分成一个或多个Partition，每个Partition对应一个文件夹
Producer：消息产生者，负责生产消息并发送到Kafka Broker
Consumer：消息消费者，向kafka broker读取消息并处理的客户端。
Consumer Group：每个Consumer属于一个特定的组，组可以用来实现一条消息被组内多个成员消费等功能。
--------------------- 
作者：wqh3520 
来源：CSDN 
原文：https://blog.csdn.net/wqh8522/article/details/79163467 
版权声明：本文为博主原创文章，转载请附上博文链接！