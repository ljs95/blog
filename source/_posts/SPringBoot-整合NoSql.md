---
title: SPringBoot-整合NoSql
date: 2019-11-19 20:27:12
tags: 
 - spring boot
 - Java
categories:
 - Java
---

# 通用配置
## maven依赖
添加Spring-Web和Spring-Security依赖，使用Spring-Security是因为使用SpringBoot的Redis依赖时，必须添加Spring-Security。在新版本SpringBoot才会这样。
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
## properties配置
8080端口指定一下，因为下面双开服务器这个配置必须在这里显示加上。
```text
server.port=8080
```

## 测试类
```java
@RestController
public class HelloController {

    @Value("${server.port}")
    Integer port;

    @GetMapping("/set")
    public String set(HttpSession session) {
        session.setAttribute("name", "johnson");
        return String.valueOf(port);
    }

    @GetMapping("/get")
    public String get(HttpSession session) {
        return (String)session.getAttribute("name") + port;
    }
}
```

<br>

# 整合Redis
## maven依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

连接redis必须要密码，否则连接不上，所以你的redis服务器必须设置密码
```text
spring.redis.host=127.0.0.1
spring.redis.database=0
spring.redis.port=6379
spring.redis.password=123456
```

启动后，浏览器打开`localhost:8080`,账号默认为user，密码在控制台打印出来了，可以去看看。页面如下：
![](https://img2018.cnblogs.com/blog/1377046/201911/1377046-20191119170225093-1131247236.png)


## Redis下的Session共享
当我们开启两个或多个Tomcat时，如何在这两个Tomcat服务中共享Session呢，而Spring直接扔个依赖给你，安装这个依赖就好了。
???????????? execute me!?
就是这么简单，添加`spring-session-data-redis`依赖就好了,如下：
```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

## 测试Session共享
使用maven使用package指令打包出来出来后（IDEA的Maven工具有package按钮，点一下就好），在target目录下可以看到你打包好的jar包，就像这样：
![](https://img2018.cnblogs.com/blog/1377046/201911/1377046-20191119170503407-520001374.png)


进入到tartget目录后，打开两个命令窗口, 分别输入以下命令:
```text
java -jar sessionhare-0.0.1-SNAPSHOT.jar  --server.port=8080 //窗口1命令
java -jar sessionhare-0.0.1-SNAPSHOT.jar  --server.port=8081 //窗口2命令
```
然后打开页面`localhost:8080`,账号默认为user，密码可以在8080的控制台看到，登录成功后，
再打开页面`localhost:8081`，你会发现不需要再次登录啦，Session共享成功！

## Nginx的负债均衡
安装Nginx可以参考我之前的文章 [Centos安装Nginx](https://colablog.cn/2019/09/02/Nginx-1/)  
如果是Ubuntu或者其他类型的系统，就依赖项不同，安装方式还是一样的。

## nginx配置
nginx配置在路径在`/usr/local/nginx/conf/nginx.conf`, 修改配置如下：
在`http`模块下修改。
```text
    upstream colablog.cn {
        server 127.0.0.1:8080 weight=1;
        server 127.0.0.1:8081 weight=1;
    }
    server {
        listen		80;
        server_name	localhost;
		
		location / {
		   proxy_pass http://colablog.cn;
		   proxy_redirect default;
	    }
    }
```
修改完nginx配置后记得要重新加载一下配置文件，修改配置文件后必须重新指定配置文件，否则启动会报错。
```text
sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf //重新指定配置文件
sudo /usr/local/nginx/sbin/nginx -s reload //重新启动nginx
```

把刚才项目打包出来的jar包扔到Linux服务器上，让程序在服务器后台运行，使用如下命令：
```text
$ nohup java -jar sessionhare-0.0.1-SNAPSHOT.jar --server.port=8080 > 8080.log &
$ nohup java -jar sessionhare-0.0.1-SNAPSHOT.jar --server.port=8081 > 8081.log &
```
打开你Linux服务器的ip地址就可以看到了。在浏览器打开`你的虚拟机ip/set`，`你的虚拟机ip/get`，重复打开几次就会发现访问不同的端口。


<br>

# MongoDb
整合MongoDb就像整合Redis那么简单，依赖和配置文件搞一下就行了
## maven依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

## properties配置
```text
spring.data.mongodb.host=127.0.0.1
spring.data.mongodb.authentication-database=admin
spring.data.mongodb.username=johnson
spring.data.mongodb.password=123456
spring.data.mongodb.port=27017
spring.data.mongodb.database=johnson
```
这样就已经配置好了，不过我们还是测试一下吧。

## 测试
`Book`实体类
```java
public class Book {

    private Integer id;

    private String name;

    private String author;

    @Override
    public String toString() {
        return "Book{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", author='" + author + '\'' +
                '}';
    }

    public Integer getId() { return id; }

    public void setId(Integer id) { this.id = id; }

    public String getName() { return name; }

    public void setName(String name) { this.name = name; }

    public String getAuthor() { return author; }

    public void setAuthor(String author) { this.author = author; }
}
```
`dao`接口
```java
public interface BookDao extends MongoRepository<Book, Integer> {
    List<Book> findBookByNameContaining(String name);
}
```
测试类
```java
@SpringBootTest
class MongoApplicationTests {
    @Autowired
    BookDao dao;

    @Test
    void contextLoads() {
        Book book = new Book();
        book.setName("colablog");
        book.setId(1);
        book.setAuthor("johnson");
        dao.insert(book);
    }

    @Test
    public void getList() {
        List<Book> all = dao.findAll();
        System.out.println(all);
        List<Book> cola = dao.findBookByNameContaining("cola");
        System.out.println(cola);
    }

    @Autowired
    MongoTemplate template;

    @Test
    public void test1() {
        Book book = new Book();
        book.setName("colablog2");
        book.setId(2);
        book.setAuthor("johnson2");
        template.insert(book);

        List<Book> all = template.findAll(Book.class);
        System.out.println(all);
    }
}
```

# 总结
文章主要是根据[江南一点雨（松哥）](https://www.javaboy.org/)总结了视频第六章内容，
代码贴的有点多，因为测试用例的关系，真是抱歉，因为有测试用例才能证明程序能走通。
好了，感谢各位的阅读，文章若有不足之处或更好的建议，请在下方留言，Thanks♪(･ω･)ﾉ。