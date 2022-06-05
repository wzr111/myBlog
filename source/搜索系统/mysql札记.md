设置为当前时间戳

ALTER TABLE index_property CHANGE createTime createTime datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP;



数据库长度设置

**MYSQL不是有65535的长度吗,为什么只有这么一点点?后来才发现,原来VARCHER的默认长度还是255,如果你想它更长,就得指定.还有,你不能指定它为65535,或是65534,这样是会错的.原因如下:**

- 存储限制。
  varchar 字段是将实际内容单独存储在聚簇索引之外，内容开头用1到2个字节表示实际长度（长度超过255时需要2个字节），因此最大长度不能超过65535。
- 编码长度限制
  - 字符类型若为gbk，每个字符最多占2个字节，最大长度不能超过32766;
  - 字符类型若为utf8，每个字符最多占3个字节，最大长度不能超过21845。
  - 若定义的时候超过上述限制，则varchar字段会被强行转为text类型，并产生warning。
- 行长度限制
  导致实际应用中varchar长度限制的是一个行定义的长度。 MySQL要求一个行的定义长度不能超过65535。若定义的表长度超过这个值，则提示
  ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. You have to change some columns to TEXT or BLOBs。

### 磁盘空间角度：

1.长度设置过长，如果有数据就会按照实际数据的长度进行设定；如果没有数据它会自己进行一定的压缩处理，不会你设定的是多长它就占多长的
1）varchar类型没有数据会默认占用4个位置，有数据就按照长度设定的占；
2）char类型和Number类型没有数据会进行压缩处理，有数据就按照长度设定的占。

### 内存角度

这几天刚解决了一个varchar 引起的性能问题，简单的就是，对于一个varchar（1000），executor在没拿到存储引擎存储的数据之前，并不会知道我这一行拿出来的数据到底有多长，可能长度只有1，可能长度是500，那怎么办呢，那就只能先把最大空间分配好了，避免放不下的问题发生，这样实际上对于真实数据较短的varchar确实会造成空间的浪费。

举例：如果我有1000个varchar（1000），但是每个只存一个字符，那么真实数据量其实只有1K，但是我却需要1M的内存去适应他。



## mysql性能调优踩坑经历

#### 字段长度





#### 连接池数量

这种问题是最经常遇到的

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2v197wdi7j21t607qacp.jpg)

主要原因是，连接池大小以及最大连接数的限制。与应用程序这边线程池的数量不对等，而单条sql执行的速度不够快，就会造成连接数过多导致mysql服务器承担不了这么多负载。

解决方案，调整最大连接数，与线程池的数量，

show status like 'Threads%' ,查看当前连接数。

druid查看当前连接数

#### 单条sql长度

涉及到发送包的长度大小限制

show variables like ‘%max_allowed_packet%’;

<img src="https://tva1.sinaimg.cn/large/e6c9d24egy1h2v14cf3d6j20n40kw0uo.jpg" style="zoom:50%;" />

查看更多参数可以这么来，

<img src="https://tva1.sinaimg.cn/large/e6c9d24egy1h2v15rg073j20ha0sgwhb.jpg" style="zoom:50%;" />



修改参数： set global max_allowed_packet = 102410141024;

最大为1G，最小为1K

这种方式有个问题，就是设置的最大连接数只在mysql当前服务进程有效，一旦mysql重启，又会恢复到初始状态。因为mysql启动后的初始化工作是从其配置文件中读取数据的，而这种方式没有对其配置文件做更改。

但是一般永久性修改：

修改配置文件

vim /etc/my.cnf 斜杠输入/查找相关属性修改即可。

#### 中途连接中断

The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.

这种现象有两种情况，一是数据库的配置有问题，查看yml配置是否正确，一般出现在刚启动时，一般很容易发现，可能性比较小

另一种情况就是，由于数据库回收了连接，而系统的缓冲池不知道，继续使用被回收的连接所致的。**这种情况比较多**

第一种解决办法，就是将mysql回收空闲连接的时间变长，mysql默认回收时间是8小时，可以在mysql目录下的my.ini中增加下面配置，将时间改为1天。

单位是秒，最大好像是24天：

​     [mysqld]

​     wait_timeout=86400

第二种解决办法，可以通过配置，让缓冲池去测试连接是否被回收，如果被回收，则不继续使用，以dbcp为例：

#SQL查询,用来验证从连接池取出的连接
          dbcp.validationQuery=SELECT 1
          #指明连接是否被空闲连接回收器(如果有)进行检验，如果检测失败，则连接将被从池中去除
          dbcp.testWhileIdle=true
          #在空闲连接回收器线程运行期间休眠的时间值,以毫秒为单位，一般比minEvictableIdleTimeMillis小
          dbcp.timeBetweenEvictionRunsMillis=300000
          #在每次空闲连接回收器线程(如果有)运行时检查的连接数量，最好和maxActive一致
         dbcp.numTestsPerEvictionRun=50
          #连接池中连接，在时间段内一直空闲，被逐出连接池的时间(1000*60*60)，以毫秒为单位
          dbcp.minEvictableIdleTimeMillis=3600000

#### 适应场景

- 如果应用程序是不间断运行的，则使用第一种情况比较省事，

- 如果为了灵活配置和安全，可以使用第二种。
- 通过测试，单核机器上效果很差，甚至不如单线程效率高。多核CPU效果提高较为明显。

#### 应用程序端的写入

线程池的核心线程与最大线程数量的设置

#### 疑问

1. 代码有没有正确释放connection
2. 数据库连接池配置的连接数太少，不够支撑我们的项目
3. 数据库连接池拿到的连接在不用的时候，状态都是sleep的。为什么会不使用呢
4. 但是项目中DruidDataSource确实是在不停的重复创建。很奇怪的现象
5. druid监控sql不出现 
   ![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2vrz854uhj20yc090wes.jpg)
   dataSource没设置filters: stat,wall,log4j2
6. 
7. 



#### 目前解决了哪些问题

1. 之前mysql连接数过多，是因为我的代码封装有问题，新建了过多的连接池，dataSource应该是单例的，不能重复创建
2. 但是项目中确实是在不停的重复创建。很奇怪的现象

#### 明晰了哪些源码

1. DruidDataSource的init过程
   1. 判断inited状态，这样确保init方法在同一个DataSource对象中只会被执行一次。（后面有加锁）。
   2. 之后内部开启要给ReentrantLock。这个lock调用lockInterruptibly。 
   3. 如果获取不到lock,则会将当前的线程休眠。
   4.  再次检测inited状态。如果为true,则返回。这里做了一个DoubleCheck。 定义initStackTrace ，为后续需要getInitStackTrace方法使用。
   5. 生成DruidDataSource的id。这是一个AtomicInteger，从1开始递增，每个DataSource都会加1。

### 调优记录

八万条数据，

| 线程池配置 core maxt queue | cost(ms) | batch/条 |
| -------------------------- | -------- | -------- |
| 2 2 2                      | 9040     |          |
| 10 10 20                   | 10243    |          |
| 9 10 1                     | 5703     |          |
| 10 10 10                   | 5806     |          |
| 10 10 1                    | 6353     |          |
| 30 40 10                   | 4931     | 1000     |
| 10 100 10                  | 3511     | 1000     |
| 10 100 10                  | 11879    | 100      |
| 10 100 10                  | 957      | 2000     |
| 20 100 10                  | 1006     | 2000     |
| 20 1000 10                 | 1000     | 2000     |
| 10 10 10                   | 3825     | 2000     |
| 10 1000 40                 | 376      | 2000     |

结论，影响最大的因素的batch的批次数，一次插入2000条，性能有显著提升。

最大线程数到了100以上即提升不大

核心线程数超过了cpu核心数，提升也很有限



#### 问题复盘

1. 问题描述
   需要把70万条数据多线程插入到数据库中，数据不能丢失，可以乱序写入。要求速度要足够快

2. 初始设计方案，

   - 多线程加载到内存list中，
   - mysql最大连接数max_xonnection = 2000, 连接池数量30，保活时间：3000ms，利用自己写的工具合成sql，10条一批次
   - Java线程池配置，core200，最大300， 队列100

3. 出现的问题

   1. Communications link failure

   2. 行数据量过大

   3. 字段内容过长

   4. 字段中有非法字符，有单双引号

   5. list中含有空元素，多线程加载到内存时，有空元素加入

   6. Data source rejected establishment of connection,  message from server: "Too many connections"

   7. show processlist

      1. <img src="https://tva1.sinaimg.cn/large/e6c9d24egy1h2vq7pkel5j211a0rwn2d.jpg" style="zoom:33%;" />
      2. 好家伙，怎么这么多连接，即使是连接池，也应该只拿到10个连接才对，现在直接整了32个。
      3. 为什么创建了153个连接，按理说1个数据库连接池拿到10个连接后，就会一直使用这10个，不会再申请了，真相只有一个！就是创建了多个数据库连接池！！

   8. 连接数过多，导致应用程度挂了，提示没有datasource set
      查看了一下mysql的线程。ps：网上的博客真的懂吗，每一个说到位的。

      - Threads_cached 。

        - cached命中率
        - 线程cache命中率=Threads_created/Connections，cache命中率当然越大越好，如果命中率较低，可以考虑增加thread_cache_size。

      - Threads_connected 活跃连接数，和 CPU 的核数是相关的。

        - Thread_connected当前打开的连接数。

      - Threads_created

        - connection能存活的最大时间。 

          原来设置60s 就会close不用的连接，后面如果有需求连接过来，还需要重新新建连接，这样可能1分钟 一批连接被杀掉，再建新的连接，很浪费资源

      - Threads_running

        - 活跃连接数，和 CPU 的核数是相关的。
        - 活跃会话数是指正在干活的数量，这个数量不是越多越好，我们要保证活跃会话要尽可能少，这样的话，mysql 才能提供最高的一个性能。
        - Threads_running官方的说法是“没有sleep的线程数”。顾名思义是：在DB端正在执行的客户端线程总数。
        - Server端保持这些连接同时客户端等待回复。有些线程可能消耗CPU或者IO，有些线程可能啥也没做单纯等表锁或行锁释放。
        - 当DB执行完这个线程，客户端收到回复，线程的状态就会从"running" 变成 "connected".

      - max_connections 

        - 允许同时连接DB的客户端的最大线程数。如果客户端的连接数超过了max_connections,应用就会收到“too many connections”的错误。
        - 这个参数实际起作用的最大值（实际最大可连接数）为16384，即该参数最大值不能超过16384，即使超过也以16384为准
        - 增加max_connections参数的值，不会占用太多系统资源。系统资源（CPU、内存）的占用主要取决于查询的密度、效率等；

4. 



慢sql记录

http://localhost:8778/druid-monitor/druid/sql.html