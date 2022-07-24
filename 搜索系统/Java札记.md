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

![image-20220602222504049](搜索系统/Java札记.assets/image-20220602222504049.png)

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



| 序号 | 键     | ArrayBlockingQueue                               | LinkedBlockingQueue                                          |
| :--- | :----- | :----------------------------------------------- | :----------------------------------------------------------- |
| 1    | 基本的 | 它由数组支持                                     | 它由链接列表支持                                             |
| 2    | 有界   | 它是有界数组队列。因此，一旦创建，容量将无法更改 | 它是无限队列                                                 |
| 3    | 通量   | 它的吞吐量比链接队列队列低                       | 链接队列比基于阵列的队列具有更高的吞吐量                     |
| 4。  | 锁     | 它使用单锁双条件算法                             | 它具有putLock用于在队列中插入元素，以及takeLock用于从队列中删除元素 |

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



@AllArgsConstructor 生成所有有参构造



---

记一次多线程调优的过程

1.class Country implements Comparable{  
2.privateint countryId ;  
3.@Override
4.publicint compareTo(Object o) {
5.        Country country = (Country)o ;  
6.returnthis.getCountryId()>country.getCountryId()?1:this.getCountryId()==
7.                country.getCountryId()?0:-1 ;  
8. }  

此时Country就具备了比较的能力了，如同Integer，String类一样，此时Country对象的集合就可以通过Collections.sort(),
或者Arrays.sort()，来进行排序了。

抽象类就是不能使用new方法进行实例化的类，即没有具体实例对象的类。抽象类有点类似“模板”的作用，目的是根据其格式来创建和修改新的类。

对象不能由抽象类直接创建，只可以通过抽象类派生出新的子类，再由其子类来创建对象。

保证线程安全的有三种方式：原子类、volatile、锁。

1.管道：管道本质是内核中维护的一块内存缓冲区。

2.命名管道：因为无名管道只适用于具有亲缘关系的线程，所以就衍生出来命名管道，这样就可以实现了非亲缘关系进程之间的通信。

3.信号：一种通知机制。

4.消息队列：是一个消息链表，既可以读消息，也可以写消息。5.共享内存：多个线程共享一片内存区域。

6.内存映射：就是将磁盘文件数据映射到内存，通过修改内存就能修改磁盘文件。7socket接口，socket接口一般用于不同主机上进程之间的通信

JAVA IO流

- 字符流 Reader 、 Writer
- 字节流 InputStream 、 OutputStream byte[]数组

三个类加载器

BootStrap ClassLoader:被称为启动类加载机制，是Java类加载层次中最顶层的类加载器，负责加载JDK中核心类库。

Extension  ClassLoader：被称为扩展类加载器，负责加载Java的扩展类库，Java虚拟机的实现会提供一个扩展目录，该类加载器在此目录里面查找并加载Java类

AppClassLoader：被称为系统类加载器，负责加载应用程序classpath目录下的所有jar和class文件。一版来说，Java应用的类都  是由它来完成加载的。可以通过ClassLoader.getSystemClassLoader()来获取它。

java栈、程序计数器、本地方法栈都是线程私有的，线程生就生，线程灭就灭

所以几个地方的内存分配和回收是确定的，不需要管。

但是java堆和方法区则不一样，我们只有在程序运行期间才知道会创建哪些对象，所以这部分内存分配和回收是动态的。一般说的垃圾回收也是针对这部分的。

1、标记-清除(Mark-sweep)：分为两个阶段，标记和清楚。

2、复制（copying）: 此算法把内存空间划分为两个相等的区域，

3、标记-整理（Mark-Compact）:此算法结合了“标记-清除”和“复制”两个算法的有点。

4、分代收集算法：这是当前商业虚拟机常用的垃圾收集算法。

如果划分？将对象按其生命周期的不同划分成：年轻代（Young Generation）、老年代（Old Generation）、持久代（Permanent Generation）。

其中持久代主要存放的是类信息，所以与java对象的回收关系不大，与回收信息相关的是年轻代，老年代。

