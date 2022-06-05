**实践能力的重要性**

fastJson序列化的变量必须要有get、方法才会展示

这取决于你使用的CPU，操作系统，其他进程正在做的事情，你使用的Java的版本，还有其他的因素。我曾经见过一台Windows服务器在宕机之前有超过6500个线程。当然，大多数线程什么事情也没有做。一旦一台机器上有差不多6500个线程(Java里面)，机器就会开始出问题，并变得不稳定。
以我的经验来看，JVM容纳的线程与计算机本身性能是正相关的。

我的16G内存支持4000个线程

```java
public static <T, E> void sampleMethod(T[] array, E ele )
```

java传入多个泛型参数

public static <E> E getValue(Object obj, Class<E> type) {

**泛型类代码 :**

```java
public class Student<T> {

    private String name;
    private int age;
    /**
     * 该数据的类型未知
     *  使用泛型表示 , 运行时确定该类型
     */
    private T data;

    public Student(String name, int age, T data) {
        this.name = name;
        this.age = age;
        this.data = data;
    }
}
<String> 指明
// 指定 泛型类 的泛型为 String 类型
        //      那么在该类中凡是使用到 T 类型的位置 , 必须是 String 类型
        //      泛型类的 泛型声明 , 使用时在 类名后面 声明
        Student<String> student = new Student("Tom", 16, "Cat");
        String data = student.getData();
// 如果不 指明泛型类型
        //      则 泛型类型 默认为 Object 类型
        Student student1 = new Student("Tom", 16, "Cat");
        // 获取的 泛型类型的变量也是 Object 类型 , 需要强转为 String
        String data1 = (String) student1.getData();

```



### **泛型方法**

```java
public class Show {
 
    public static  <T> T show(T t) {
        System.out.println(t);
        return t;
    }
 
    public static void main(String[] args) {
        // 返回值不用强转，传进去是什么，返回就是什么
        String s = show("一个优秀的废人");
        int num1 = show(666);
        double num2 = show(666.666);
        System.out.println("------------------------");
        System.out.println(s);
        System.out.println(num1);
        System.out.println(num2);
    }
}
```





java对象转JSONObject、JSONObject转java对象及String转

```java
JSONObject jsonObj = (JSONObject) JSONObject.toJSON(javaBean); 

Student stu = JSONObject.parseObject(jsonObj, Student.class);

JSONObject jsonObj = JSON.parseObject(str); //str为
```



### jdbcTemplate连接池怎么用

普通的jdbc没有办法做到高并发多连接，

使用报错案例

Data source rejected establishment of connection,  message from server: "Too many connections"

### Druid数据库连接池

​    这个是阿里巴巴提供的数据库连接池，可能是世界上最好的数据库连接池了吧。

理论上来讲，配置了maxActive：20，最大的连接数上限就是20了。但实际并不是

可以确认，确实超过了[druid](https://so.csdn.net/so/search?q=druid&spm=1001.2101.3001.7020)的连接池上限。

```java
Connection conn = dataSource.getConnection();
Statement statement = null;
ResultSet resultSet = null;
statement = connection.createStatement();
resultSet = statement.executeQuery(sql);
```



1.标准接口： DataSource javax.sql
         方法：
           获取链接：getConnection():



```
@Bean(name)在jvm启动的时候就会初始化
```

![image-20220602222504049](Java%E6%9C%AD%E8%AE%B0.assets/image-20220602222504049.png)

## 线程池

<img src="https://tva1.sinaimg.cn/large/e6c9d24egy1h2v3vgeqimj20u01dcadd.jpg" style="zoom: 33%;" />

从这张图可以看出线程执行完任务不会退出的原因，runWorker内部使用了while死循环，当第一个任务执行完之后，会不断地通过getTask方法获取任务，只要能获取到任务，就会调用run方法，继续执行任务，这就是线程能够复用的主要原因。

线程池状态具体是存在ctl成员变量中，ctl中不仅存储了线程池的状态还存储了当前线程池中线程数的大小

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

**线程池的关闭**

线程池提供了shutdown和shutdownNow两个方法来关闭线程池。

shutdown方法

![img](https://pic4.zhimg.com/50/v2-6fed2e658bebe1c25f29ebf34bd0ab38_720w.jpg?source=1940ef5c)

就是将线程池的状态修改为SHUTDOWN，然后尝试打断空闲的线程（如何判断空闲，上面在说Worker继承AQS的时候说过），也就是在阻塞等待任务的线程。

shutdownNow方法

![img](https://pic2.zhimg.com/50/v2-ee2a9bce58f74f422c810f6958bbdafa_720w.jpg?source=1940ef5c)



就是将线程池的状态修改为STOP，然后尝试打断所有的线程，从阻塞队列中移除剩余的任务，这也是为什么[shutdown](https://www.zhihu.com/search?q=shutdown&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2471908876})Now不能执行剩余任务的原因。

**执行shutdownNow会丢失任务**

**线程池的监控**

在项目中使用线程池的时候，一般需要对线程池进行监控，方便出问题的时候进行查看。线程池本身提供了一些方法来获取线程池的运行状态。

```java
# 注意，必须要用这个类创建，ExecytorSearvice接口不支持
ThreadPoolExecutor threadPoolExecutor
  
```

- getCompletedTaskCount：已经执行完成的任务数量
- getLargestPoolSize：线程池里曾经创建过的最大的线程数量。这个主要是用来判断线程是否满过。**可用于调优**
- getActiveCount：获取正在执行任务的线程数据
- getPoolSize：获取当前线程池中线程数量的大小

1）线程数

线程数的设置主要取决于业务是IO密集型还是CPU密集型。

CPU密集型指的是任务主要使用来进行大量的计算，没有什么导致线程阻塞。一般这种场景的线程数设置为CPU核心数+1。

IO密集型：当执行任务需要大量的io，比如磁盘io，网络io，可能会存在大量的阻塞，所以在IO密集型任务中使用多线程可以大大地加速任务的处理。一般线程数设置为 2*CPU核心数

java中用来获取CPU核心数的方法是：Runtime.getRuntime().availableProcessors();

### Java线程池系列之execute和submit区别

1. execute只能提交Runnable类型的任务，没有返回值，而[submit](https://so.csdn.net/so/search?q=submit&spm=1001.2101.3001.7020)既能提交Runnable类型任务也能提交Callable类型任务，返回Future类型。

2.  execute方法提交的任务异常是直接抛出的，而submit方法是是捕获了异常的，当调用FutureTask的get方法时，才会抛出异常。

3. > execute是Executor接口的方法，而submit是ExecutorService的方法，并且ExecutorService接口继承了Executor接口。