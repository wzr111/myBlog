
## flume项目，apache顶级项目
分布式高可用、高可用对海量日志采集、聚合和传输的系统，存储到中心化数据存储系统。

flume的springboot启动

- pom加依赖 logback-flume-appender_2.10
- 配置logback.xml
  - 输出级别
  - 日志文件名格式
  - 保存时间
  - 记录的级别level
  - flumeAgents：[localhost:44444] 
- /conf/xxx.conf 配置 name
- 启动 -name a1, 

flume近似实时的推送，并且可以满足数据量

## 为什么要使用flume
分布式、可靠、可用的系统，有效的监听应用、收集、聚合（channel）、移动（sink）大量的日志数据，

从许多不同的源（sink下一跳）（不同的应用皆有可能有日志输出）到一个集中的数据存储

Apache Flume的使用不仅限于日志数据聚合，由于数据源是可定制的（channel 节点名称 + 端口） 

flume可用传输大量的事件数据，包括但不限于网络流量数据、社交媒体生成但数据、等任何可能的数据源。

flume的设计宗旨是向Hadoop等分布式集群批量导入基于事件的海量数据。

sink的最后一跳一般是HDFS，hbase、hive、Kafka等众多支持分布式存储的系统中。

flume事件被定义为具有字节有效负载和一组可选字符串属性的数据流单元

事件暂存在每个代理的通道中（channel，可类比Kafka的topic），将事件传递到流中的下一个代理或终端存储哭（HDFS）。

只有在事件存储到下一个代理的通道或终端存储哭中之后，才会从通道中删除事件，保证流之间，端到端的可靠性。

source和sinks分别封装在事务中存储或检索，

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h3m5j632lij20t20e00t9.jpg)

### Event

事件是flume内部数据传输的最基本单元，将传输的数据进行封装，事件本身是由一个载有数据的字节数组和可选的heard儿头部信息构成，如下图所示。

flume以事件的形势将数据从源头传输到最终的目的地。

### agent

source、channel、sink，将事件流从一个外部数据源收集并发送给下一个目的地。

#### source

#### channel
一种短暂的存储容器，先进先出的队列，sink去消费。

1. memory channel，队列容量为可存储最大事件数量，适用于高吞吐量场景，在agent出现错误时可能会丢失部分数据
2. file channel，基于文件系统的持久化存储
3. Kafkachannel 存储在Kafka集群中

#### sink
获取channel暂时保存的数据并进行处理，sink从channel中移除事件，并将其发送给下一个agent，简称下一跳活着事件的最终目的地，比如hdfs

### 数据流模型
![](https://tva1.sinaimg.cn/large/e6c9d24ely1h3m5qjhgr0j20wk0gkabh.jpg)

##### 入门使用案例

使用flume监听某个端口，使用netcat向这个端口发送数据，flume将接收到的数据打印到控制台。

netcat是一款tcp/UDP测试工具，
>yum install -y nc

>nc lk localhost port # 服务端
> nc localhost port 客户端

必须设置的值

- source

channels、type、bindIp、port type为netcat

- channel

type

- logger sink

channel、type
```
# example.conf 单节点flume配置
# 定义agent名称为a1
# 设置3个组件的名称
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# 配置source类型为NetCat，监听地址为本机，端口为44444
a1.sources.r1.types = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# 配置sink类型为logger
a1.sinks.k1.type = logger

#配置channel类型为内存，内存队列最大容量为10000，一个事务中从source接受的events数量或者发送给sink的events数量最大为100
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# 将source和sink绑定到channel
a1.sources.r1.channels = c1
# 消息存储在c1上
a1.sinks.k1.channel = c1
```

>bin/flume-ng agent -n a1 -c conf/ -f conf/example.conf -Dflume.root.logger=INFO,console

flume占用44444端口，不能用nc -lk localhost 44444 来。

#### 数据持久化
使用组件
- file Channel

type
- checkpointDir   检查点文件存放路径
- dataDirs      日志存储路径，多个路径使用逗号分隔，使用不同点磁盘上点多个路径能提高file channel的性能

企业中应用程度会写很多日志， tail -f app.log

可以用flume进行采集，发送到hdfs

tail只能监控一个命令，还有重复消费的问题。


host拦截器，可以记录消息是从哪个主机过来的。

时间拦截器

header key value的拦截器


通道选择器

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3mrujoofwj20yy0jcq55.jpg)

搜集到的日志大概长什么样。

flume spark

# spark
一次性数据计算，框架在处理数据的时候，会从存储设备中读取数据，进行逻辑操作，然后将处理的结果重新存储到存储介质中

上一次计算的结果交给下一次作业操作，io交互的次数比较多，计算的效率较低。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3msjwzd8ij20yw0gaabg.jpg)

- spark计算的结果放于内存中，不过容易造成内存不足的情况。
- spark多个作业之间数据通信是基于内存，而Hadoop是基于磁盘的。spark和Hadoop的根本差异是多个作业之间的数据通信问题。
- spark Task的启动时间快。spark采用fork线程的方式，而Hadoop采用创建新的进程的方式。
- spark只有在shuffle的时候将数据写入磁盘，而Hadoop中多个MERGE作业之间的数据交互都要依赖磁盘交互。
- spark的缓存机制比hdfs的缓存机制更高效

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3msk5z1eyj20yg0dm75f.jpg)


spark适用于内存有优势的机器中，在世纪生产环境受内存的限制，可能会由于内存资源不够导致job执行失败，

此时，MapReduce其实是一个更好的选择。所以spark并不能完全替代MR

## spark模块
### Apache Spark Core 核心模块
Spark Core中提供了Spark最基础与最核心的功能，Spark其他的功能如：Spark SQL，Spark Streaming，GraphX，， MLlib都是在Spark Core的基础上进行扩展的

### Spark SQL
操作结构化数据的组件。
### Spark Streaming
spark平台上针对实时数据进行流氏计算的组件，提供了丰富的处理数据流的API
### Spark MLlib
机器学习算法库
### Spark Graphx
面向图计算提供的框架与算法库

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3mt7daa4lj20we0h8dhe.jpg)

离线画像业务介绍

文章内容标签化：内容标签化、根据内容定性的制定一系列标签。。

这些标签可以是描述性、如类型、关键词等，针对于文章就是文章相关等内容词语

应用到电影上就是，类型、地区、叙事、标签（如生化危机）？

- 关键词：TextRank计算出的结果Topk个词以及权重
- 主题词：TEXTRANK的TOPK词 与TFIDF计算的TOPK个词的交集

所有历史文章tdidf计算

原始文章数据的合并，

通过spark sql进行操作。合并三张表的内容，写入到hive中，

- article存放文章计算结果

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h3peu7hc83j20f707ydg8.jpg)

**词语与词频统计**

>from pyspark.ml.feature import CountVectorizer

**总词汇大小。文本中必须出现的次数**

>cv = CountVectorizer(inputCol="words", outputCol="CountFeatures", vocabSize=200*10000, min)

训练词频统计模型

>cv_model = cv.fit(words_df

>cv_model.write().overwrite().save("hdfs_address"))

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h3pfmhg8hzj20jq0apgn1.jpg)

> from pyspark.ml.feature import CountVectorizerModel
> cv_m = CountVectorizerModel.load("hdfsURL")

cvmodel，CountVectorizerModel可以先把词分出来，得到word_list

IDF模型，获得所有分词的逆文档频率
> from pyspark.ml.feature import IDF
> idf = IDF(inputCol = "countFeature", outputCol="idfFeature")
idfModel = idf.fit(cv.result)

得到了两个少量文章测试的模型，以这两个模型为例子查看结果

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h3pj1pwq22j20fe091mxj.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h3pj27kuqij20cy09u74r.jpg)

index为词在文档的位置，根据index和cvmodel生成的词表word_list可以获取具体的关键词

IDF再对Cvmodel结果进行计算TFIDF结果

分成1000个词，对每一个词进行排序筛选，取出topK

总结： 整个结构
模型为什么耗时，因此进行大量的文本运算。
1. 推荐架构和业务流
   1. 用户行为收集，业务数据收集，留意表设计
   2. 批量计算(离线计算)：用户文章画像，留意拓展到电影
   3. 用户的召回结果，排序精选过程 ？
   4. grpc的实时推荐业务流的搭建
      1. 缓存
2. 开发环境 python环境
   1. hive集群部署
   2. python容器部署
3. 数据库迁移
   1. sqoop迁移mysql业务数据到hive表。五张表
      1. mysql每天新记录插入，以及修改操作，需要同步。
      2. sqoop全量导入与增量导入。导入时间记录，incremental，lastmodify最后更新时间。两值不同即为增量更新。
      3. user_profile
         ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h3ppwnhzpqj20f4097gms.jpg)
      4. 
   2. 增量更新，
      1. append 插入
      2. lastmodify，时间戳，唯一标识的实体id，merge。
4. 收集用户日志
   1. flume -> hive表
   2. flume开启，监听端口
   3. 建立分区-hive表，防止影响性能，关联路径
   4. 埋点参数设计 .
      > 就是在应用中特定的流程收集一些信息，用来跟踪应用的使用状况。后续用来进一步优化产品或者提供运营的数据支撑。
      1. 阅读停留时间 read
      2. 点击事件 click
      3. 曝光时间 exposure 
      4. 收藏事件 collect
      5. 分享事件 share
      6. 埋点参数文件结构
         1. 曝光的参数：下拉刷新，推荐新的若干篇文章
         2. 我们将埋点参数设计成一个固定格式的json字符串
      7. supervisor工具管理，启动监听flume日志收集程序。
         1. 、
   5. 用户数据收集user_action。
      1. PM项目经理、算法推荐工程师一起指定埋点需求文档
      2. 后端、客户端APP集成
      3. 推荐人员基于文档埋点测试与梳理。
   6. 离线部分，将flume收集到的服务器A的日志持久化到Hadoop服务器hdfs的hive中
      1. hive数据库的设置、分区
      2. 手动关联分区的Hadoop目录
      3. 收集到新的数据库中
   7. flume收集日志配置，channel、sink
5. 离线画像业务 关键词为TextRank计算的TopK个词以及权重。
    > 主题词 TextRank的TopK词与TFIDF计算的TopK个词的交集
   1. 关键词以及权重
      1. TFIDF 训练cv model 拿到分词list，和IDF Model，word在文档中的重要程度
         > 理解tfidf含义：评估一个词对于一个文件集对于其所在文档的重要程度
         > 字词的重要性随着他在文中出现的次数成正比增加，但同时会随着它在语料库中出现但频率成反比下降。
         > tf = n/N IDF = lg[D/(1+d)] n为在doc出现的次数，N为doc总字数。 D为语料库所有的文档总数，d表示语料中出现某个词的文档数量。lg以10为底。
         > 最终TFIDF计算 = TF * IDF
         > 弱化常见词，保留重要的词
      2. 中间表tfidf-keywords表
      3. 拿到关键词和其对应的分数,存入keyword-value（重要程度）表
   2. textrankApi的使用
      1. 使用的jieba.textrank
      2. 先理解文章画像。就是给每篇文章定义一些词，主题词与关键词最大的区别就是主题词经过了规范化处理
      3. 关键词：文章中一些词的权重高的。原理；jieba.analyse.TextRank
         1. 通过词之间的相邻关系构建网络，然后用PageRank迭代计算每个节点的rank值，啥意思？
            1. 每个点就是每个词
            2. 图模型提取关键词，提取步骤如下
               1. 文本T按照完整句子进行分割，对于每个句子，根据关键词词典，进行分词和词性标注处理（jieba）即可，并过滤掉停用词（词典）
               2. 得到保留后的候选关键词
               3. 建立关键词图G=(V,E)，K窗口。其中V为节点集，由上一步生成的候选关键词组成，然后采用共现关系构造任两点之间的边，当他们对应的词汇在长度为K
                  1. 理解K窗口。在分词过程中另一个词离它比较近，则给他投票。
                  2. K为9，前后五个身位的关键词对他进行投票经过排序的结果可知，这两个数获得的票数最多，因此这两字比较重要。其他的词都是为它服务的。
                     ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h3po7cf4bkj20l00coq4h.jpg)
                     ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h3po8f2k9dj208y07s3yk.jpg)
                  3. 
            3. 计算权重排序。
         2. 再谈PageRank算法（Google），一个网页被很多其他网页链接到说明这个网页比较重要，就是它的PageRank值会比较高
         3. 那么如果一个pagerank比较高的网页链接到一个其他的网页，被链接到的网页的PageRank值也会相应提高。
         4. 简而言之，重要性是可以在图结构中传播的。
      4. 
   3. 关键词为TextRank计算的TopK个词以及权重。
   4. 主题词 TextRank的TopK词与TFIDF计算的TopK个词的交集

### 文章画像计算
步骤：
1. 加载UDF，保留关键词以及权重计算（textRank*IDF） article_profile
   1. topK个关键词作为画像结果。如果命中实体列表，加权。
2. 合并关键词权重到字典结果
   1. TFIDF计算出的索引定位到关键词
3. 将tfidf和textrank共现的词作为主题词（取交集，join一下）
4. 将主题词表和关键词表进行合并，插入表

### 文章相似度
目标
- 知道文章向量计算方式
- 了解word2Vec模型原理
- 知道文章相似度计算方式

需求： 
- 首页频道推荐： 每个频道推荐的时候，会通过计算两两文章相似度，快速达到在线推荐的效果。
- 比如用户点击文章，我们可以将离线计算好相似度的文章排序快速推荐给该用户。
- 此方法可以解决冷启动的问题，冷启动，新用户，新app，只能推荐热门文章。发生一个新行为，立马推荐相似文章

方式：
- 计算两两文章TFIDF之间的相似度
- 计算两两文章的word2Vec或者doc2Vec向量相似度


关键词以及权重 textrank*idf权重

主题词 textrank和tfidf的共取topK的交集

离线增量更新：固定时间更新

如何计算海量文章相似结果：LSH+文章向量（word2Vec)
- word2Vec
  - 词的独热表示，阿里巴巴对商品使用对
  - 词的分布式表示，一种低维实数向量，让相关或者相似的词在距离上更近了。
分频道进行计算每个频道中的一些文章相似度。

文章词向量：
1. 根据频道内容，读取不同频道号，获取相应频道数据并进行分词
2. spark Word2Vec训练保存模型

根据频道内容，读取不同频道号，获取相应频道数据

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h4152z9s8qj20yh0fzjt1.jpg)
 

例如频道[
    linux,
    程序员，
    搜索推荐
]

10万篇1000字文章，训练word2Vec需要半天时间，25个频道 要训练很久。

模块
>from pyspark.ml.feature import Word2Vec

API 
> class pyspark.ml.feature.Word2Vec(vectorSize=100, minCount=5, numPartition=1,stepSize=0.025,maxlter=1,seed=None,
> inputCol=None,)
LSH

无监督学习比如word2Vector 十年前的产物 自己生成标签、关键词、

BERT最新的模型，nlp任务的基石

n-gram模型，定义，n个相邻字符构成的序列。

特点，统计型、简单、泛化能力差、无法得到单词的语义信息
- unigram 一个单词
- bigram 两个单词
- trigram 三个单词

用途最先用于垃圾邮件分类。

特征唯独随着语料词汇增大和n增大而指数增大。

单词的语义表征 
- one-hot encoding，词表构成的数组，获得单词在数组的位置index

分布式
- 类似word embedding， 浮点型的向量，位置更加精准

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h45bpvcvalj20iy0dl0tg.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h45bxshvhrj20fo0923z2.jpg)

主题内容画像：

[user_id, action_time, article_id, shared, clicked, collected, exposure, read_time]

用户画像存在Hbase中，方便与快速读取使用，离线分析也想要使用我们可以建立HIVE到HBase到外部表。

> create 'user_profile', 'basic', 'partial', 'env'

Hbase支持原子性更新。

基于内容推荐流程：
1. 建立物品画像
   1. 用户打tag
   2. 电影分类值[悬疑，科幻，] 与tag[牛逼] 合并
   3. 根据电影打id把tag和分类值合并起来，求tf-idf
   4. 根据tf-idf打结果，为每一部电影筛选出来打top-n个关键词 [movie_id, [悬疑，科幻, 牛逼]]
   5. 电影ID-关键词-关键词权重 [film_123, 英雄, 3]
   
   TF-IDF特征提取技术：物品画像的特征标签，主要都是指的是电影的导演、演员、编剧作者、发行公司、也就是他们的特征提取，尤其是特征向量的计算是比较简单的
   
如直接给作品的分类定义0或者1的状态。

但另外一些特征，比如电影但内容简介、电影的影评、简介摘要等文本数据，这些被称为非结构化数据，首先他们本应该也属于物品的一个特征标签，

但是这样的特征标签进行量化时，也就是计算它的特征向量时是很难去定义的。

因此这时就需要借助一些自然语言处理、信息检索技术，将如用户的文本内容信息的非结构化数据进行量化处理，从而实现更加完善的物品画像、用户画像

TF-IDF算法便是其中一种在自然语言处理领域中应用比较广泛的一种算法。可用来提取目标文档中，并得到关键词用于计算对于目标文档的权重，并将这些权重组合到一起得到特征向量。

介绍算法原理





3. 建立倒排索引
   1. 通过关键词找到电影 [悬疑，[杀人回忆:3, 追击者:10]]
   2. 遍历电影id-关键词-关键词权重数据，读取每一个关键词，用关键词作为key [film_id,tfidf]作为value保存到dict {"英雄": [film_123, 4]}
4. 用户画像
   1. 看用户看过哪些电影，到电影到电影id-关键词-关键词权重，数据中，找到电影所对应的关键词
   2. 把用户看过的所有关键词放在一起，统计词频，每个词出现的次数 tag可能是多个用户给其打的标签
   3. 出现次数多的关键词，作为用户的兴趣词，整个兴趣次实际上是用户画像的关键词
   4. 看过的movie_id: [111,145,188] 每个电影都有profile，几个电影合并兴趣词。获得兴趣词的权重排名
   
5. 根据用户的兴趣次，找到兴趣词对应的电影，多个兴趣词可能对应一个电影{电影id：[关键词权重1，关键词权重2]}
   1. 把每一部电影对应的关键词权重求和
   2. 每一部电影提取关键词。！！**派上用场了。标签、描述、别名、名称。**
6. 产生推荐结果。
   1. 候选集，[user_id, [兴趣词list]]
   2. 拿兴趣词去一个表里面，取出相关的电影。related_movies [movie_id, related_weight兴趣词权重] 找出相应的兴趣词权重最高的
   3. [user_id, [word, 权重]] 只考虑用户的兴趣程度。
   4. 每一个关键词求和排序，从高往低推荐给用户。
   5. 可以两者相乘，得到两个结果。

word2Vec 2013年谷歌推出的开源nlp工具，特点是开源将所有的词向量化，这样词与词之间就可以定量

一个词表示一个向量 分布式 [0.99,0.05,0.01] [年份,类型，热度] 三个维度。

三维，还可以降为，如年份*热度变成一个维度。

相似度通过余玄？


两个重要模型，CBOW 把一个词从词窗剔除，在CBOW下给定n词围绕着词w，word2vec预测一个句子中其中一个缺漏的词c，

CBOW 

即以概率p(c|w)来表示。相反地，skip-gram给定词窗中的文本，预测当前的词。

原理：拥有差不多上下文的两个单词的意思往往是相近的。

词向量

用向量来表示词语，可以表示语义层面的含义

如果用word2Vec模型创建的词向量，两个词向量相似度比较高。说明这两个词是近义词。gensim word2Vec模型训练词向量模型。

word2Vec设定模型参数，setVectorSize,setWindowSize, setNumIterations

参数观察上下文关系的窗口长度。

训练模型时需要保留的词语出现的频率，iter=20 迭代20个词。

找到topn

文档向量

训练模型并保存Doc2vec 通过向量来表示一篇文档，一篇文档就对应一个电影。

向量相似度，代表来电影的相似程度 （可以离线训练存入redis）

[film_id1: [film_other1, film_other2, film_other3]]

[film_id2: [film_other7, film_other12, film_other23]]

....

传入电影的某些标签[可以考虑什么标签]，找到电影文档所对应的向量。

通过doc2vec找到传入的向量 最相似的n个向量，每个向量代表来一部电影。

doc2Vec是建立在word2Vec上的，用于直接计算以文档为但愿的文档向量，这里我们将一部电影的所有标签词，作为整个文档，这样可以计算出**每部电影的向量**

通过计算向量之间的距离，**来判断用于计算电影之间的相似程度**。

合过影的概念很好解释，时间窗口

from gensim import

model = Doc2Vec(documents, vectoor_size=100, window = 3, min_count=1, worders=4, epochs=20)

这样解决物品冷启动问题。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h45u8hb41bj20fc08jwf0.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h45uanl1g3j20f906igm3.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h45udbszrvj20fe0cbt9w.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h45uff2idnj20f80b3gm3.jpg)


