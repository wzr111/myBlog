
## 关于bbs系统建表设计

### 方案一：
发帖主题和回复信息存放在一张表，并在这个表中添加user_name字段。

对数据库的操作而言，检索数据的性能基本不会对数据造成非常大的影响。（非模糊查询）

而对表与表之间的连接却会产生巨大的影响。特别在有巨量数据的表之间？为啥

因此对问题对定位基本能够确定：在显示和检索数据时，尽量降低数据库对连接以及表与表之间的连接：

| 比较项    | RESTful    | RPC    |
  | ---- | ---- | ---- |
|    通信协议  | HTTP     |   一般基于tcp   |

## 方案二
水平切分。 三张表

1。用户信息表

简单用户信息表

user_id,user_name, 头像，email,homepage,tel,add..

具体用户信息表

索引：id，user_id, 名字，地区，

2。帖子主题表

id,user_id,topic_id,title,content截断,....点击次数

索引：id, user_id, topic_id,title,content截断

设计理由：因为大部分的操作都是进入首页，所以只需要提供主题的索引，不必加载全部，

用户想看哪个帖子，点进去再读一次数据库，**延时也是可以接受的**。

缓存top100的帖子到redis中，新增到帖子也可以新增到redis中，方便计算点击次数

每天凌晨定时任务持久化到mysql表中，更新top100。

另外主题id和回复表分开存储有利于以后的分库分表操作。分库分表一般按照年份、分类分。

分库分表涉及到分布式各个数据库表的id自增问题，造成id冲突，我的系统好像没有这方面的需求。

每天论坛訪问量300万左右，更新帖子10万左右。 
每月数据在10W*30=300W. 可按月分表
每年帖子在300W*12=3600W, 推算数据不会小于30T. 可按年分库


3。主题回复表

id,topic_id,user_id,title,content,....

与top_id绑定，

回复有点赞、取消点赞、

主题更新次数多，对于mysql是很大的压力，如果每回复一次都更新mysql，

redis数据结构设计：以主题id为key，hash结构 
楼层表设计

```json
[
  {
    "floor_id_1": {
      "user_id": "123",
      "user_name": "kk大神",
      "avatar": "/home/data/avatar",
      "reply_list": ["reply_id_1","reply_id_2"],
      "floor_num": "自增",
      "reply_content": "优秀",
      "点赞次数": 2
    }
  },
  {
    "topic_id_2": {
      "user_id": "124",
      "user_name": "kk大神2",
      "avatar": "/home/data/avatar",
      "reply_list": ["reply_id_1","reply_id_2"],
      "floor_num": "自增",
      "reply_content": "优秀plus",
      "点赞次数": 2
    }
  }
]
```

楼层内回复设计
```json
[
  {
    "reply_id_1": {
      "user_id": "123",
      "user_name": "kk大神",
      "avatar": "/home/data/avatar",
      "reply_id": "123",
      "floor_id": "floor_id_1",
      "father_reply_id": "reply_id_0 可为空，空则不显示父评论信息", 
      "reply_content": "优秀",
      "点赞次数": 2
    }
  },
  {
    "reply_id_2": {
      "user_id": "124",
      "user_name": "kk大神2",
      "avatar": "/home/data/avatar",
      "reply_id": "xxx",
      "floor_id": "自增",
      "father_reply_id": "reply_id_0 可为空，空则不显示父评论信息",
      "reply_content": "优秀plus",
      "点赞次数": 2
    }
  }
]
```

用户点击论坛首页：

1. 加载top100主题帖
2. 点击某个主题贴，
   1. 根据topic_id，加载topic_id详细信息，其中有reply_list[reply_id_1, reply_id_2 ...]
   2. 根据[reply_id_1, reply_id_2 ...]，加载主要信息，默认加载两条
   3. 拿着reply_id_1去加载楼层内的详细信息
   4. 最后结构如下
```json
{
  "top_id": "xxx",
  "topic_content": "帖子内容",
  "floor_list": [
    {
      "floor_id_1":{
        "user_id": "124",
        "user_name": "kk大神2",
        "avatar": "/home/data/avatar",
        "reply_list": ["reply_id_1","reply_id_2"],
        "floor_num": "自增",
        "reply_content": "优秀plus",
        "点赞次数": 2,
        "reply_detail": [
          {
            "user_id": "123",
            "user_name": "kk大神",
            "avatar": "/home/data/avatar",
            "reply_id": "123",
            "floor_id": "floor_id_1",
            "father_reply_id": "reply_id_0 可为空，空则不显示父评论信息",
            "reply_content": "优秀",
            "点赞次数": 2
          },
          {
            "user_id": "123",
            "user_name": "kk大神",
            "avatar": "/home/data/avatar",
            "reply_id": "123",
            "floor_id": "floor_id_1",
            "father_reply_id": "reply_id_0 可为空，空则不显示父评论信息",
            "reply_content": "优秀",
            "点赞次数": 2
          }
        ]
      }
    },
    {
      "floor_id_1":{
        "user_id": "124",
        "user_name": "kk大神2",
        "avatar": "/home/data/avatar",
        "reply_list": ["reply_id_1","reply_id_2"],
        "floor_num": "自增",
        "reply_content": "优秀plus",
        "点赞次数": 2,
        "reply_detail": [
          {
            "user_id": "123",
            "user_name": "kk大神",
            "avatar": "/home/data/avatar",
            "reply_id": "123",
            "floor_id": "floor_id_1",
            "father_reply_id": "reply_id_0 可为空，空则不显示父评论信息",
            "reply_content": "优秀",
            "点赞次数": 2
          },
          {
            "user_id": "123",
            "user_name": "kk大神",
            "avatar": "/home/data/avatar",
            "reply_id": "123",
            "floor_id": "floor_id_1",
            "father_reply_id": "reply_id_0 可为空，空则不显示父评论信息",
            "reply_content": "优秀",
            "点赞次数": 2
          }
        ]
      }
    }
  ]
}

```

最后落表mysql

- topic_table
- floor_table
- reply_table

因为mysql索引只对于查询有帮助，更新的性能是非常慢的，所以每日回复的记录先更新到redis中，凌晨再持久化到mysql
## 表设计需要考虑的问题
- 检索数据的难易程序
- 加载数据的成本
- 后期数据的存储成本
- 尽量降低数据库的连接以及表与表之间的连接

索引对于 增删改操作的性能影响比较大。

所以，索引个数越多，对于insert操作来说，维护的成本就越大，插入一条数据的速度也就越慢。

把数据插入索引的过程中，为了维护索引中字段的顺序，会先在索引中查找这个值

如果能找到，就把这个值查到后面空闲的地方，如果没有找到，就先把值加入到叶子节点

然后在分支节点中新增这个值 和 指向叶子节点的指针（就是一个地址）。

**内存页** 在这个过程中，如果某个页满了，还要新申请一个空的页，把满的页拆分开，把一半的索引数据放到空闲页中，而且为了保证数据的一致性

（这个插入操作是并发的，可能有几十上百个线程同时进行），会给相关的索引页加上闩锁（一种更低级别的内存锁）。

delete操作刚好和isnert相反，当删除一条数据时，会把这条数据涉及到的多个索引中的数据删除。

### 提快速度的关键
1. 建立索引并在查询时充分利用;
2. 避免使用关联,这样避免整表扫描;使用关联不如多次使用主键查询来的快;
3. 一些处理增减运算的功能尽可能放到内存中来做,比方组织主题和回复;
