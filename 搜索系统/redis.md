

### 为什么zset要使用跳表

1）内存占用更少，自定义参数化决定使用多少内存

2）有序集合很多时候用zrange 或者zrevrange 。性能至少不比平衡树差
>Set zSet = redisTemplate.opsForZSet().rangeByScore("zSet", 1, 5); 
>获取指定score区间的值

3）简单更容易实现和维护

### redis的key上限的问题
一个Redis实例最多能存放理论上2的32次方个keys（40亿），

并且在实际中进行了测试，每个实例至少存放了2亿5千万的keys

举例说明，现需要搭建Cluster集群6个节点，redis的端口号依次为7000，7001，7002，7003，7004，7005，6个节点分为3组，一主(master)一从的形式

计算每组的哈希槽排序，每组=16384/3=5461

那么：

第一组端口7000 的哈希槽为 0-5460

第一组端口7001 的哈希槽为 5461-10922

第一组端口7002 的哈希槽为 10923-16383

Cluster集群搭建

1， 第一步准备好8个redis.conf文件

linux环境下，redis部署在/application目录下，先创建好好8个目录执行如下命令：

```yml
cd /application
mkdir rediscluster
cd rediscluster/
mkdir redis7000
mkdir redis7001
mkdir redis7002
mkdir redis7003
mkdir redis7004
mkdir redis7005

后面做动态增加到哈希槽

mkdir redis7006
mkdir redis7007
```

```yml
dbfilename 7000dump.rdb
daemonize yes #后台启动
protected-mode no ; ## 允许外部访问
port 7000 #修改端口号，从7000到7007
cluster-enabled yes #开启cluster，去掉注释
cluster-config-file 7000nodes.conf #自动生成
cluster-node-timeout 15000 #节点通信时间
logfile  /application/rediscluster/redis7000/redis.log"/application/rediscluster/redis7000/redis.log"
```
注意上面带7000-7007 是端口号，每个文件夹对应放到对应的端口号，

例如：redis7005 文件夹里面的redis.conf内容对应

```yml
dbfilename 7005dump.rdb
port 7005 #修改端口号，从7000到7007
cluster-config-file 7005nodes.conf #自动生成
logfile  /application/rediscluster/redis7005/redis.log"/application/rediscluster/redis7005/redis.log"
```
![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3kd320juej208807zt8v.jpg)

```shell
/application/redis/bin/redis-server /application/rediscluster/redis7000/redis.conf
/application/redis/bin/redis-server /application/rediscluster/redis7001/redis.conf
/application/redis/bin/redis-server /application/rediscluster/redis7002/redis.conf
/application/redis/bin/redis-server /application/rediscluster/redis7003/redis.conf
/application/redis/bin/redis-server /application/rediscluster/redis7004/redis.conf
/application/redis/bin/redis-server /application/rediscluster/redis7005/redis.conf
```
第三步，分配hash槽
>/application/redis/bin/redis-cli --cluster create  172.16.166.122:7000  172.16.166.122:7001  172.16.166.122:7002  172.16.166.122:7003  172.16.166.122:7004  172.16.166.122:7005  --cluster-replicas 1

![m表示主s从](https://tva1.sinaimg.cn/large/e6c9d24egy1h3kd4jfibmj20go08e40d.jpg)

第四步，查看分配的hash槽

>172.16.166.122 是内网IP
/application/redis/bin/redis-cli -h 172.16.166.122 -p 7000 -c

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3kd6gbnibj210004l40r.jpg)

可以看到有3组master，**只有master才能分配哈希槽**，集群会自动进行选举，如下图7000，7001，7002是master节点，7000的从节点是7004，

可能有些朋友按照上图搭建7000的子节点是7003，都有可能的，按照选举结果为准

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h3kd8rciodj20kl044q33.jpg)

任何hash、list、set、和sorted set都可以放2的32次方个keys（40亿）个元素。
### redis项目中用的数据结构

#### set
spop 随机弹出一个元素

smembers获取所有元素，返回的结果是无序的（随机的）

sinter求多个集合的交集 

sunion求多个集合的并集

sdiff求多个集合的差集

将交集、并集、差集 的结果保存
```shell
sinterstore destination key [key ...] 
suionstore destination key [key ...] 
sdiffstore destination key [key ...]
```

#### zset 有序集合




