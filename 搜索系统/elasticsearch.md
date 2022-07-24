# 字段

对于object对象，一个字段可以嵌套多个字段

```json
{
	"properties": {
		"age": {
			"type": "integer"
		},
		"name": {
			"properties": {
				"first": {
					"type": "text"
				},
				"last": {
					"type": "text"
				}
			}
		}
	}
}
```



ES支持的数据类型，

1. 字符串类型

   - text ，

     - 适合全文索引，

     - 可以切词，

     - 不用于排序，很少用于聚合，适合长文本

       text⽂本数据类型，⽤于索引全⽂值的字段。使⽤⽂本数据类型的字段，它们会被分词，在索引之前将字符串转换为单个术语的列表(倒排索

       引)，分词过程允许ES搜索每个全⽂字段中的单个单词。什么情况适合使⽤text，只要不具备唯⼀性的字符串⼀般都可以使⽤text。

   - keyword

     - 只能通过精准匹配

     - 适合用于过滤、排序、聚合

     - keyword关键字数据类型，⽤于索引结构化内容的字段。使⽤keyword类型的字段，其不会被分析，给什么值就原封不动地按照这个值索
       引，所以关键字字段只能按其确切值进⾏搜索。什么情况下使⽤keyword，具有唯⼀性的字符串，例如：电⼦邮件地址、MAC地址、⾝份证号、状态代码...等等。

       

2. 整数类型。在满足需求的情况下，尽可能选择范围小的数据类型，字段的长度越短，索引和搜索的效率越高。

   1. long
   2. integer
   3. short
   4. byte

3. 浮点类型

   1. double：64位双精度IEEE 754浮点类型
   2. float：32位单精度IEEE 754浮点类型 
   3. half_float：16位半精度IEEE 754浮点类型 
   4. scaled_float ： 缩放类型的的浮点数 

4. 逻辑类型

   1. boolean

5. 日期类型

   1. date日期格式字符串
   2. 时间戳，毫秒级

6. 范围类型。范围类型要求字段值描述的是一个数值、日期或IP地址的范围， 在添加文档时可以使用： gte、gt、lt、lte分别表示 >=、 > 、< 、<= 。

   1. range
      1. integer_range
      2. float_range
      3. long_range
      4. double_range
      5. data_range
      6. ip_range

7. 二进制类型。[二进制](https://so.csdn.net/so/search?q=二进制&spm=1001.2101.3001.7020)字段是指用base64来表示索引中存储的二进制数据，可用来存储二进制形式的数据，例如图像。默认情况下，该类型的字段只存储不索引。二进制类型只支持index_name属性

   1. binary

8. 复合类型

   1. 数组类型 array 
      对象类型 object ：JSON格式对象数据不需要特殊配置，**⼀个字段如果被配置为基本数据类型，**就是天⽣⽀持数组类型的。任何字段都可以有0个或多个值，但是在⼀个数组中数据类型必须⼀样。
   2. 嵌套类型 nested 
      地理类型 地理坐标类型 geo_point 
      地理地图 geo_shape 
      特殊类型 IP类型 ip 
      范围类型 completion 
      令牌计数类型 token_count 
      附件类型 attachment 
      抽取类型 percolator 

9. 多数据类型

10. 有些字段可能会以不同的方式进行检索，如果文档字段只以一种方式编入索引，检索性能就会受到影响。所以针对text和keyword，es专门提供了一个配置字段多数据类型的参数fields，它可以让一个字段同时具备两种数据类型的特征。

    ```json
    PUT articles{
        "mappings"{
            "properties":{
                "title":{
                    "type":"text",
                    "fields":{
                        "raw":{"type":"keyword"},
                        "length":{
                                                    "type":"token_count", 
                              "analyzer":"standard"
                        }
                    }
                }
            }
        }
    }
    ```

    title字段被设置为text，同时通过fields参数又为该字段添加了两个子字段raw和length，且分别为keyword类型和token_count类型。使用fields设置的子字段，在添加文档时不需要不需要单独设置字段值， 他们与title共享相同的字段值， 只是会以不同的方式处理字段值， 且在查询时不会展现出来。

    ```json
    PUT demo2
    {
        "settings": {
            "number_of_shards": 4,
            "number_of_replicas": 1
        },
        "mappings": {
            "properties": {
                "id": {
                    "type": "long"
                },
                "title": {
                    "type": "text"
                },
                "university": {
                    "type": "object",
                    "properties": {
                        "name": {
                            "type": "text"
                        },
                        "foundingYear": {
                            "type": "date"
                        },
                        "address": {
                            "type": "text"
                        }
                    }
                }
            }
        }
    }
    ```

      

es一个字段可以定义多个分词器

```json

```



拼音分词器

首先配置tokenizer，它的工作是把”2018世界杯”切分成全拼和首拼，一共就2个term。

### tokenizer

```json
"tokenizer" => 
[ 
    // 拼音分词 
    "name_py_tokenizer" => 
        [ "type" => "pinyin",   "keep_none_chinese" => false, // 对非中文不拆分词 
        "keep_full_pinyin" => false, // 关闭: 刘德华 -> liu, de, hua   
        "keep_joined_full_pinyin" => true, // 刘德华 -> liudehua 
        "keep_none_chinese_in_joined_full_pinyin" => true,  // 刘德华2016 -> liudehua2016   
        "keep_first_letter" => true, // 刘德华 -> ldh 
        "keep_none_chinese_in_first_letter" => true, // 刘德华2016 -> ldh2016   
        "none_chinese_pinyin_tokenize" => false, // 没有卵用 
        ]
 ]
```

如果我们要索引”世界杯worldcup”，那么keep_none_chinese=true则会导致把非中文部分拆出1个独立的term：worldcup，这不是我们希望的，我们只需要shijiebeiworldcup或者sjbworldcup，所以这个选项需要设置false。

如果我们要索引”世界杯worldcup”，那么keep_none_chinese=true则会导致把非中文部分拆出1个独立的term：worldcup，这不是我们希望的，我们只需要shijiebeiworldcup或者sjbworldcup，所以这个选项需要设置false。

### filter

接着，需要利用ES内置的edge-ngram filter，把全拼和首字拼按前缀拆term，也就是上面我们列举的大量的前缀term：

Tokenizer 后续深度研究 [分词器对比](https://blog.51cto.com/u_11440114/5100691)

standard 按单词进行分割
letter 按非字符类进行分割
whitespace 按空格进行分割
UAX URL Email 按 standard 分割，但不会分割邮箱和 url
NGram 和 Edge NGram 连词分割
Path Hierachy 

### 批量查询

- 批量查询的好处

  > 比如说要查询100条数据，那么就要发送100次网络请求，这个开销是很大的。如果进行批量查询的话，查询100条数据，就只要发送1次网络请求，网络请求的性能开销缩减100倍。
  > -一条一条查询

- 操作

  ```json
  GET /_mget
  {
    "docs":
    [
      {
        "_index":"test_index",
        "_type":"test_type",
        "_id": 1
      },
      {
        "_index":"test_index",
        "_type":"test_type",
        "_id": 2
      }
    ]
  }
  #返回
  {
    "docs": [
      {
        "_index": "test_index",
        "_type": "test_type",
        "_id": "1",
        "_version": 1,
        "found": true,
        "_source": {
          "test_field": "test1"
        }
      },
      {
        "_index": "test_index",
        "_type": "test_type",
        "_id": "2",
        "_version": 1,
        "found": true,
        "_source": {
          "test_field": "test2"
        }
      }
    ]
  }
  ```

  [批量查询demo](https://blog.csdn.net/weixin_43064185/article/details/102629711)



插入不同类型的文档。

在Elaticsearch 6.x版本中已经只允许一个索引下只有一个*type*,声明多个*type*已经标记为过期,但是仍可以使用。



## 初始化配置

[setting设置写入](https://blog.csdn.net/qq_16504067/article/details/94624593)

### 1. 索引刷新间隔调整 

refresh_interval

默认情况下索引的refresh_interval为1秒,这意味着数据写1秒后就可以被搜索到,每次索引的 refresh 会产生一个新的 [lucene](https://so.csdn.net/so/search?q=lucene&spm=1001.2101.3001.7020) 段。

这会导致频繁的 segment merge 行为,如果你不需要这么高的搜索实时性,应该降低索引refresh 周期。

（即你可能想优化索引速度而不是近实时搜索， 可以通过设置 refresh_interval ， 降低每个索引的刷新频率）

```json
PUT /my_logs
{
  "settings": {
    "refresh_interval": "30s"  //每30秒刷新 my_logs 索引
  }
}

PUT /my_logs/_settings
{
  "refresh_interval": -1  //关闭自动刷新
}
PUT /my_logs/_settings
{
  "refresh_interval": "1s" //每秒自动刷新
}
```



### 2. 事务日志操作间隔调整：translog flush

  translog是[elasticsearch](https://so.csdn.net/so/search?q=elasticsearch&spm=1001.2101.3001.7020)的事务日志文件，它记录了所有对索引分片的事务操作（add/update/delete），每个分片对应一个translog文件。

translog是用来恢复数据的。类似binlog。

Es用“后写”的套路来加快写入速度 — 写入的索引并没有实时落盘到索引文件，而是先双写到内存和translog文件，有三种状态（可搜索 & 未落盘 & 已写日志） 。

如果掉电，es重启后还可以把数据从日志文件中读回来。

默认设置下,translog 的持久化策略为:每个请求都flush.对应配置项为:

> index.translog.durability: request

这是影响 es 写入速度的最大因素.但是只有这样,写操作才有可能是可靠的,原因参考写入流程。

如果系统可以接受一定几率的数据丢失,调整 translog 持久化策略为周期性和一定大小的时候 flush:

> index.translog.durability: async *//异步刷新*
>
> index.translog.sync_interval: 120s *//间隔120s异步刷新（设置后无法更改）*
>
>
> index.translog.flush_threshold_size: 1024mb *//内容容量到达1gb异步刷新*



### 3. **segment merge**

#### 什么是segment

> segment merge 操作对系统 CPU 和 IO 占用都比较高。
>
> segment merge是什么，es底层是由lucene实现，每个索引都是一个lucene文件，每个索引包含多个shard和多个replica。每个lucene文件又多个segment组成。

#### segmen merge的意义

如果数据segment过多，索引每次查询时会**加载全部segment信息到内存**，要查询全部的segment导致查询效率降低。

同时如果有大量删除的文档存在，磁盘空间不能释放，实际索引文件会很大。分片分配、分片均衡的时候会浪费大量资源（IO，带宽，CPU等）导致集群yellow。

**所以segment数量越少，索引效率越高。**



### 批量插入

>插入文档操作的一种优化，因为每次插入单条文档，都会向es中发送请求。然后es执行在返回结果；
>
>如果有大批量的文档数据需要插入，这个时候单挑插入操作显然是不合理的；
>
>之前学习的[命令行](https://so.csdn.net/so/search?q=命令行&spm=1001.2101.3001.7020)批量执行方式：
>
>POST /_bulk

bulk request会加载到内存里，如果太大的话，性能反而会下降，因此需要反复尝试一个最佳的bulk size。一般从1000-5000条数据开始，尝试逐渐增加。另外，如果看大小的话，最好是在5-15MB之间。

---

es目录结构

data文件夹，生产环境文件夹必须修改，包含索引、元文件、文档等。可以用软链吗？

jdk，目录，

单个项目启动多个节点。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h2xdc485kkj20pm0coq4a.jpg)

就是拷贝一样的安装包，分别启动

![image-20220605145522180](../../../Library/Application%20Support/typora-user-images/image-20220605145522180.png)



![image-20220605145656253](../../../Library/Application%20Support/typora-user-images/image-20220605145656253.png)



![image-20220605145724180](../../../Library/Application%20Support/typora-user-images/image-20220605145724180.png)



<img src="../../../Library/Application%20Support/typora-user-images/image-20220605200425702.png" alt="image-20220605200425702" style="zoom:25%;" />

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h2xuht3265j21c00dyta4.jpg)

ES集群内部荣宰机制

master节点，候选节点，不存数据。

学会如何会用，面临调优的时候，才是考验的时候

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h2xvdkg0f1j21fl0u00vg.jpg)

每天前一百人可以免费观看vip视频，增加高并发锁的内容

mysql的索引查看

tablename.frm 表文件

tablename.ibd innodb搜索引擎

mysql的datapage是16kb，每次io都是16kb，b+树减少io次数。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h2xwe2i0jqj214w0p6jt5.jpg)

每一台es的节点都是负载均衡的节点

复合查询、聚合分析、排名、地理位置分析

处理PB级数据

应用：百度谷歌全文索引、电商垂搜、数据分析、数据挖掘

data log在生产环境下不能放在环境目录，因为在更新的时候会覆盖重写。

每一个节点就是一个Java进程

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2yt1s5x10j217k0retbi.jpg)



增加节点或者挂点节点，会触发node balance

es的节点跟hdfs不太一样，hdfs会有namenode节点和datanode节点。namenode存储元数据。

es都存数据，是个对等性架构。