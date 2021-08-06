---
title: 盘点我从无到有搭建一个Springboot遇到的坑
date: 2021-07-31 16:46:33
categories: 
- Spring Boot
tags:
- java
- 框架
tags:
thumbnail: https://tva1.sinaimg.cn/large/008i3skNgy1gt0a709vq7j318z0u043r.jpg
---



以前的学校的时候，搭建项目，配置环境一度让我很头疼，只能用别人写好的现成的代码，这次从项目的创建，各个分层的代码、到application.yml、数据库的连接，最终到我网址输入一条增加命令，更新到了数据库的过程

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt08enthd4j30m40c3q3j.jpg)



post模拟写入请求：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt08d8i4d9j31ce0pi40t.jpg)



## 坑盘点

### 整个流程不清楚

写代码就没有办法有清晰的思路，例如本次请求： http://localhost:8080/user/add

### URL

传到服务器，会选择本地端口号，因为本地服务器可能运行了多个项目，到了8080之后，还需要选择项目名称，这里在application配置了

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt08hfbbijj310y0l4aeg.jpg)



之前写配置文件的时候，context-path是需要写在servlet里面的，我没写servlet导致一直报错，

 http://localhost:8080+/ 之后，是找到了项目开始的地方，

### controller

去controller匹配我后面的路径

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt08l9yooqj31ai0ka0yj.jpg)



http://localhost:8080/user/add 分发到了adduserRestful方法，这个方法需要调用服务层，

### service

即

userService.addUer(user)

这里又有个坑，adduserRestful()方法里面不能直接填入User user参数，因为前端给的数据是json文件，而不是一个user类，需要@RequestBody User user把对象解析出来，这里附上User的图

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt08q8l01vj31gs0u0n3g.jpg)

这里逐一解释那些注解，

- @Entity注解，在javax.persistence包中，使用@Entity注解去定义JPA中的实体，说明这个**表在数据库中**是可以一一映射的。
- @Table注解用来说明该实体类对应数据库中哪个类，name即数据库中表的名字
- Getter和Setter注解等价于@Data注解，相当于一种注入方法吧，GetID之类的方法即不用写了，写了好像会报错
- GeneratedValue是主键自增，括号内自增的方式也是必不可少，**如果用了这个注解，数据库中也要选取主键并且要设置自增方法与此匹配**！否则报错。
- @Column就是类属性和数据库中字段相匹配，name即是数据库中的名字
- @Id注解是说明这个属性是注解，默认从1开始，post不写这个字段都可以，会自增

请求体中提交的json：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt08z66fkgj30yk0j40ue.jpg)

json本质上是一个字符串，变成现成的类对象给

```java
@PostMapping("/add")
    public void addUserRestful(@RequestBody User user){userService.addUser(user);}
```

需要解析出来，

### domain

这个过程是由user类的注解协助完成的

再返回给addUserRestful，然后调用service，

![image-20210731171750135](/Users/wuzhenren/myblog/source/_posts/盘点我从无到有搭建一个Springboot遇到的坑.assets/image-20210731171750135.png)



@Service注解是告知，服务层在这里，

 要说明@Service注解的使用，就得说一下我们经常在spring配置文件applicationContext.xml中看到如下图中的配置：

```xml
<!-- 采用扫描 + 注解的方式进行开发 可以提高开发效率，后期维护变的困难了，可读性变差了 -->
<context:component-scan base-package="com.study.persistent" />
```

在applicationContext.xml配置文件中加上这一行以后，将自动扫描指定路径下的包，

如果一个类带了@Service注解，将自动注册到Spring容器，不需要再在applicationContext.xml配置文件中定义bean了，

类似的还包括@Component、@Repository、@Controller。

```java
@Service("courseDAO")
@Scope("prototype")
public class CourseDAOImpl extends HibernateDaoSupport implements CourseDAO{
......
}
```

 其作用就相当于在applicationContext.xml配置文件里配置如下信息：

```xml
<bean id="courseDAO"
      class="com.study.persistent.CourseDAOImpl" scope="prototype">
      ......    
</bean>
```

如果Service括号的参数不写的话，默认是跟类名字是一样的，

```java
@Service

public class UserServiceimpl implements UserService {
```



写了的话，相当于给这个类起了个别名，

### 交给controlled->service

json解析完成后，返回给

```java
@PostMapping("/add")
    public void addUserRestful(@RequestBody User user){userService.addUser(user);}
```



service层的方法

```java
@Override
    @Transactional(rollbackFor = Exception.class)
    public void addUser(User user) {
        userApiRepository.save(user);
    }
```

### 写入数据库

userApiRepository是继承了JPA的方法的，有save方法。

```java
public interface UserApiRepository extends JpaRepository<User,Integer>, JpaSpecificationExecutor<User> {
}
```

通过获得的user类，JPA将数据save到数据库中，怎么实现的详情了解Jpa

最终这条记录在数据库中躺着

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt09oj8gmrj30mw09p3z4.jpg)



