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

