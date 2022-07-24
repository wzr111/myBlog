---
title: 项目部署Kafka踩坑记录
date: 2022-06-18 21:04:56
categories:
tags:
thumbnail: https://tva1.sinaimg.cn/large/e6c9d24egy1h3eil0n8tdj21900u0nby.jpg
---

zookeeper单机部署三个节点，一直启动失败，原因需要在每个data节点文件中写入一个myid文件。
echo 1 > dir/data/zk1/myid

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3cqxe1j8qj216q0o6tcr.jpg)
查看所有topic
>bin/kafka-topics.sh --bootstrap-server 9.135.76.106:9092 --list

创建topic first
>bin/kafka-topics.sh --bootstrap-server 9.135.76.106:9092 --topic first --create --partitions 1 -- replication-factor 3

查看topic详情
>bin/kafka-topics.sh --bootstrap-server 9.135.76.106:9092 --topic first --describe

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3cr5ihfpmj20o604nt96.jpg)

bin/kafka-topics.sh --bootstrap-server 9.135.76.106:9092 --topic first --alter --partitions 3
![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3cro7zt2sj20os05w0t7.jpg)
分区只能增加，不能减少。

>bin/kafka-console-producer.sh --bootstrap-server 9.135.76.106:9092 --topic first
向分区发送消息

接收消息
>/usr/local/kafka]# bin/kafka-console-consumer.sh --bootstrap-server 9.135.76.106:9092 --topic first

从初始开始消费
>bin/kafka-console-consumer.sh --bootstrap-server 9.135.76.106:9092 --topic first --from-beginning

>sh bin/zkCli.sh 进入zookeeper客户端

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3cy3eeqkej20ti02y0sx.jpg)

下载prettyzoo，客户端，比较方便查看信息

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3cy5m9d7cj21eg0ssn46.jpg)

zookeeper选举过程

1。broker在zookeeper里面注册，新服务服务只需要配置cnfig文件即可

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3dkw4t31hj20bv02f74a.jpg)

新服务服役，不会立即同步分片，需要进行负载均衡操作。

创建一个要均衡的主题，

> vim topics-to-move.json
> {"topics": [{"topic":"first"}], "version": 1}

生成一个负载均衡的计划

> bin/kafka-reassign-partitions.sh -- bootstrap-server host:9092 --topics-to-move.json --broker-list "0,1,2,3" --generate

创建副本存储计划
> vim increase-replication-factor.json

退役旧节点 流程类似

Kafka概述
一、 Kafka定义

分布式发布订阅的消息队列，消息队列

发布订阅，消息分为多种类型（topic），订阅者根据需求，选择性订阅

最新定义

流平台（存储、计算）

二、消息队列应用场景

1）缓存削峰

2）解耦

3）异步通信

三、两种模式

1）点对点

一个生产者对应一个消费者

2）发布订阅

多个生产者 消费者多个  而且相互独立，多个topic，不会删除数据

四、架构

1）生产者

2）broker，

1。就是一个节点服务器的概念。

2。topic主题对数据分类，电影、电视剧、订单etc

3。分区

4。可靠性，副本

5。生产者、消费者，只针对leader操作

3）消费者

1）消费者之间相互独立

2）消费者，某个分区只能由一个消费者消费，因为是类似流式处理，消费一次
4）zookeeper

1）broker.ids 0 1 2 3

2) leader信息

三、生产者

1。原理

2。异步发送API

四、分区

1。好处
- 存储 存储海量数据，提高储存上限，通过元数据管理，保证可靠性
- 计算 提高生产者到消费者的并行度
- 默认分区规则
  - 指定分区
  - 指定key，key的hashcode值，对分区数求模
  - 没有指定key，先随机，粘性分区，但是必须保证跟上一个不相同。即random到上一次到结果，继续random
- 自定义分区，实现partitioners接口

五、吞吐量提高

1）批次大小，32k，16k

2）linger.ms 5-100ms 缓存

3）压缩

4）缓存大小32m 64m

六、可靠性

acks 
- 0 可能会丢失数据
- 1 也可能丢 传输普通日志
- -1 完全可靠，副本大于等于2，isr大于等于2，可能有数据重复的问题

七、数据重复

1）幂等性

<pid, partition, seqNumber> 幂等性默认打开

2）事务

底层基于幂等性

生产者会先申请一个pid，

然后向leader发送数据，

commit申请，持久化到topic主题里面

提交成功之后，会提示已经持久化到硬盘了，

事务协调器会校验是否真的持久化，

七、数据有序

1）单分区内有序（有条件）

2）多分区有序，需要排序

九、乱序

1）kafka 1.x ，缓存请求infight = 1 

2）新版本，没有幂等性，inflight=1

3）有幂等性，可能会重发倒致后到。类似计算机网络的滑动窗口

四、broker

1）zk存储了哪些信息

(1) broker.ids

(2) broker.leader 活跃节点信息

(3) broker.controller 监听ids的变化

监听到ids的变化之后，zookeeper就会组织重新选举，从leader拉取最新的活跃节点信息

在isr中选择活跃的，并且在ar中排在前面的节点作为新的leader。

3）服役和退役

1）先创建一个均衡的主题，

2）形成一个均衡的计划

3）执行计划

4）验证计划

五、Kafka副本

默认副本一个，生产环境一般为2个。

Follower故障处理细节

所有的follower从leader拉取消息。

leader最先收到消息，通常leader的消息多一些

LEO log End Offset：每个副本的最后一个offset，LEO其实就是最新的offset+1

高水位线，HW High Watermark，所有副本中最小的LEO，可想像成鸽巢原理的短板。

Kafka文件存储机制

topic是逻辑的概念，作为分为的。而partition才是物理上的概念。

每个partition对应一个log文件，即存储的producer文件数据。

而为了防止单个log文件过大导致数据定位效率地下，

Kafka采用分片和索引机制，将每个segment分为多个segment。

每个segment包括index文件，log文件，存在topicname + partition号。如first-0

Kafka日志默认保存七天，含有
- baseOffset 基准offset
- lastOffset 相对offset
- **position**
- baseSequence
- lastSequence
- producerId
- size createTime 压缩等

如何在log文件中定位到offset=600的Record

对其中对position进行二分查找。

00001005.index，为稀疏索引，4kb数据才记录一条索引

文件清理策略，保存时间默认7天 log.retention.hours

默认检测时间 5分钟

### 高效读写数据
(1) kafka本身是分布式集群，可以采用分区技术，并行度高

(2) 该数据采用稀疏索引，即4kb记录一条索引，通过二分法查找快速定位到要消费的数据

(3) 顺序写磁盘

  Kafka的producer生产数据，要写入到log文件中，追加到文件末尾

segment为单位，一个是1G。顺序写，速度可以高达600M/s，而随机写只能达到100k/s

与磁盘的机械结构有关，顺序写之所以快，是因为其省去了大量的磁头寻址时间

回忆操作系统。电梯算法

#### 零拷贝
Kafka的数据加工处理操作交由Kafka生产者和Kafka消费者处理。Kafka broker应用层不关心存储的数据

**所以不用走应用层，传输效率高** 如何理解这句话？

零拷贝就是生产者数据发送到Kafka集群中，不需要关心数据是什么样到，

因为拦截器、序列化工具都是在生产者客户端和消费者端完成，

输出流到了Kafka端口，由内核完成转发过程，消费者拉取到输入流，

一切到消费者客户端处理

##### PageCache页缓存
Kafka中毒依赖底层操作系统提供的pageCache功能。

当上层有写操作时，操作系统只是将数据写入PageCache。

当读操作发生时，先从PageCache中查找，如果找不到，再去磁盘中读取。

即所谓的缺页中断，实际上pageCache是把更可能多的空闲内存都当作了磁盘缓存来使用。

## 消费者
>pull 拉取模式
> push 推送模式

Kafka没有采取push模式，为什么？

因为broker决定消息发送速率，很难适用所有消费者到消费速率。

例如推送到速度为50m/s，消费者1 2 就来不及消费，但是消息堆积甚至丢失

pull模式不足之处是，如果Kafka没有数据，消费者可能会陷入循环中，一直返回空数据

一个topic也只能由消费者组中的一个消费者消费。一个主题是绝不能由多个消费者消费的，**流式处理**。

### 消费者组原理
消费者都维护一个groupID，消费者数量不能超过主题数量 

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3e14z2boyj21c00r8q8j.jpg)

**不同消费者组之中的消费者是可以消费同一个主题的。**

#### coordinator：辅助实现消费者组的初始化和分区的分配
节点选择，groupid的hashcode值%50

例如，groupid=1，1%50=1，那么该主题的1号分区，在3号broker上，就选择这个节点的coordinator作为这个消费者组的老大

**相当于选出了一个leader节点，制定一个消费方案。**消费者组下的所有的消费者提交offset的时候就往这个分区去提交offset

每个消费者都会和coordinator保持心跳，默认3s，一旦超时45s，该消费者会被移除，并触发再平衡

或者消费者处理消息的时间过长，最大处理时间5分钟，**也会触发再平衡**













    

