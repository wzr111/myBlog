pom包的启动类启动不能扫描property

@Configuration注解和@Component注解的区别。、

从这个例子可以看出区别。

```java
public Driver driver(){
        Driver driver = new Driver();
        driver.setId(1);
        driver.setName("driver");
        driver.setCar(car());
        return driver;
    }
```



```car
@Bean
    public Car car(){
        Car car = new Car();
        car.setId(1);
        car.setName("car");
        return car;
    }
```



测试代码如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestApplicationTests {

    @Autowired
    private Car car;

    @Autowired
    private Driver driver;

    @Test
    public void contextLoads() {
        boolean result = driver.getCar() == car;
        System.out.println(result ? "同一个car" : "不同的car");
    }

}
```



我们明确有这样一个过程，car初始化，driver初始化需要初始化car，

- 用@configuration注解时，初始化driver会直接拿car的bean，会去Spring容器中直接car的bean，

所以初始化car和driver只new了一次car，即用的同一个car

- 用@Component注解时，初始化driver会初始化car两次。

总结：两个都可以当作配置类，

扫描@Configuration时，会为它的类进行cglib代理，生成当前子类Class，会对其构造方法进行拦截，第二次调用car()方法时直接从Bean Factory之中获取对象，拿到的是同一个对象。

而@Component不会为其生成cglib代理。



补充：@configuration的类在Spring启动时，被bean注解的方法和对象会被applicationContext类扫描，构建初始化容器，并且对于@Value的对象会对其进行属性填充。

说说@Value注解，**@Value必须在注册类中使用，且类加载方式必须为注入方式**。

所以普通类中不能使用@Value注解内部属性，同时普通类中也不能使用@Autowired或者@Resource注入由@Value注解的注册类（这是因为装配只能在注册类中进行）。

1. @Configuration不可以是final类型；
2. @Configuration不可以是匿名类；
3. 嵌套的configuration必须是静态类。

Spring中有两种[类型](https://www.yaoruanwen.com/tag/3109)的Bean，一种是普通Bean，另一种是工厂Bean，即FactoryBean



组件类，不同的private final DruidPool druidPool;

Configuration、Component打印的地址是一样的

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2ugc8shaxj20sc09a763.jpg)



mybatis 报错的时候才打印日志？



Spring @Value注释在构造函数中不起作用

据我所知，值注入发生在构造函数调用之后。可以将构造函数拆解

newFun()

```java
public DruidPool(){
	...
  newFun()
}

newFun(@Value("${spring.datasource.username}") String username)
```



或者

@Autowired

构造函数

```
@Autowired
public DruidPool(env){
	...
}
```

@Bean怎么取呢，我的思路是这样的

配置类A@Component，这样启动就会扫描初始化，生成待取的Bean

```java
@Bean(name = "druidDataSource")
    public DruidDataSource druidDataSource(){
      ...
    }
```



配置类B@Component 启动的时候，构造函数自动执行

```java
public DruidPool(DruidDataSource druidDataSource){
        this.druidDataSource = druidDataSource
    }
```



那么springbean初始化的时候，默认执行哪种构造函数呢

<img src="https://tva1.sinaimg.cn/large/e6c9d24egy1h2v63d6syqj21bc0q8tey.jpg" style="zoom:50%;" />



有参和无参都有时，**执行无参而不是有参**

 

---

将String或text转化为JSONObject

再将JSONObject强转为Map<String, Object>

```java
JSONObject jsonObject = JSONObject.parseObject(sourceAsString);
Map<String, Object> mp = (Map<String, Object>)jsonObject;
```

对象转为json格式的串，需要先将对象强转为JSONObject，再调用toString()



### es集群

#### 集群解决三个问题

1. 解决并发的问题，高并发
2. 解决物理存储资源上限的问题
3. 解决数据备份的问题，高可用

elasticsearch集群节点重启导致分片丢失的问题



```json
POST entity_film529/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```

query里面可以根据条件删除

获取所有索引

```console
GET /_cat/indices?v
```

删除索引

```console
DELETE /customer?pretty
```

可创建加数据一步到位

```console
PUT /customer/doc/1?pretty
{
  "name": "John Doe"
}
```



![](https://tva1.sinaimg.cn/large/e6c9d24ely1h304xw11aaj20gz06sdg7.jpg)

指定ID重复提交会更新数据，幂等性，版本更迭。

后台其实是先删除，再写入，不过指定了ID

```json
POST /customer/doc/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}
```



GET /_settings?pretty 获取所有索引的设置信息

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h31zsgdmkrj215y0lgace.jpg" style="zoom:33%;" />





“failed to obtain node locks”的原因通常是没有获得这个lock文件的操作权限，我知道的有两个原因：

（1）node.lock被其他进程使用了，这也是网上大多数的解释。解决方案呢，首先查看es的进程，然后杀掉。具体如下：

ps aux | grep elastic

kill -9 [pid]

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h30pmraa3hj20lq04dq3h.jpg)

工厂模式就是，提供这么一个接口，丢进去什么类，就生产什么类，这样减少new的代码书写，减少错误。
显示unssigned的节点。

​    [curl](https://so.csdn.net/so/search?q=curl&spm=1001.2101.3001.7020) -s "http://localhost:9200/_cat/shards" | grep UNASSIGNED 

有可能是内存的问题。

#### es自动平衡

一：关闭自动分片，即使新建index也无法分配数据分片

```json
curl -XPUT http://localhost:9200/_cluster/settings -d '{
  "transient" : {
    "cluster.routing.allocation.enable" : "none" } }'
```



二：关闭自动平衡，只在增减ES节点时不自动平衡数据分片

```json

curl -XPUT http://192.168.1.213:9200/_cluster/settings?pretty -d '{
  "transient" : {
    "cluster.routing.rebalance.enable" : "none" } }' 
```



设置完以后查看设置是否添加成功：

curl http://localhost:9200/_cluster/settings?pretty

重新启用自动分片

```json
curl -XPUT http://192.168.1.213:9200/_cluster/settings -d '{
  "transient" : {
    "cluster.routing.allocation.enable" : "all" } } 
```

延迟副本的重新分配

```json
PUT /_all/_settings
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m"
  }
}
```

未分配节点重新分配延迟到5分钟之后



这将防止Elasticsearch立即开始数据恢复，直到集群中至少有八个（数据节点或主节点）节点存在。

```css
gateway.recover_after_nodes: 8
```



GET /_cluster/settings?include_defaults=true

```json
{
  "index": "entity_film",
  "shard": 0,
  "primary": true
}
```



**对某一个实例进行重启后，很有可能会导致该实例无法找到master而将自己推举为master的情况出现，为防止这种情况，需要调整 elasticsearch.yml 中的内容**

```yml
discovery.zen.minimum_master_nodes: 2
```

这个配置就是告诉Elasticsearch除非有足够可用的master候选节点，否则就不选举master，只有有足够可用的master候选节点才进行选举。
该设置应该始终被配置为有主节点资格的点数/2 + 1，例如:
有10个符合规则的节点数，则配置为6.
有3个则配置为2.



blocked by: [SERVICE_UNAVAILABLE/1/state not recovered

failed to connect to master 

原来是ping不通本机地址

ifconfig| grep inet

重启电脑解决了



### es解决重复插入的问题

我司某个环境的es中被导入了重复数据，导致查询的时候会出现一些重复数据，所以要我们几个开发想一些解决方案，我们聊了聊，出了下面一些方案：
1.从源头解决：导入数据时进行唯一性校验
2.从数据解决：清洗数据，将重复的数据查出后清理，然后入库
3.从查询解决：查询时筛选重复数据

我就从**查询**着手，找到了聚合查询的方法

#### 聚合(AGGREGATIONS)

根据city，对twitter索引的文档进行分组
aggs：聚合
my：自定义名称
terms：根据结果分类
field：筛选字段
city：需要分类的字段



<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h32028366tj21580l2dja.jpg" style="zoom:67%;" />





### 字段权重的问题

boost 参数可以设置字段的权重。

boost 有两种使用思路，一种就是在定义 mappings 的时候使用，在指定字段类型时使用；另一种就是在查询时使用。

实际开发中建议使用后者，前者有问题：如果不重新索引文档，权重无法修改。

1. mapping 中使用 boost（不推荐）：

   > ```json
   > PUT blog
   > {
   >   "mappings": {
   >     "properties": {
   >       "content":{
   >         "type": "text",
   >         "boost": 2
   >       }
   >     }
   >   }
   > }
   > ```
   >
   > 

2. 查询的时候指定boost

   ```json
   GET blog/_search
   {
     "query": {
       "match": {
         "content": {
           "query": "你好",
           "boost": 2
         }
       }
     }
   }
   ```

   

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h32hsxcw77j217o0n0ad1.jpg)

好用的log日志

```
<dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-to-slf4j</artifactId>
            <version>2.10.0</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jul-to-slf4j</artifactId>
            <version>1.7.25</version>
            <scope>compile</scope>
        </dependency>
```




