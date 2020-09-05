---
title: SpringBoot-整合JDBC和MyBatis
date: 2019-11-18 21:44:03
tags: 
 - spring boot
 - Java
categories:
 - Java
---

# 摘要
该文章主要为记录如何在SpringBoot项目中整合JDBC和MyBatis，在整合中我会使用简单的用法和测试用例，毕竟该文章目的是为了整合，而不是教大家如何去使用。希望大家多多包涵。

# 通用配置
下面介绍的整合JDBC和整合MyBatis都需要添加的实体类和配置

## 数据库表
```sql
CREATE TABLE `user`  (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `address` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 4 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;
```

## 实体类
添加简单的User实体类，用于下面jdbc和mybatis的使用和测试。再添加一个toString方法为了测试时看结果比较简单。
```java
public class User {

    private Integer id;

    private String username;

    private String address;

    public Integer getId() { return id; }

    public void setId(Integer id) { this.id = id; }

    public String getUsername() { return username; }

    public void setUsername(String username) { this.username = username; }

    public String getAddress() { return address; }

    public void setAddress(String address) { this.address = address; }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```

## maven配置
mysql版本根据自己数据库版本设置
druid为阿里云提供的数据源（可理解为连接池）
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
    <version>8.0.18</version>
</dependency>
```

## 数据库配置
数据库properties配置肯定是少不的啦.
```text
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.username=username
spring.datasource.password=password
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/mydatabase
```

<br>

# 整合JDBC
## maven依赖
添加springboot提供的jdbc依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

## 使用
``` java
@Service
public class UserService {

    @Autowired
    JdbcTemplate jdbcTemplate;

    public Integer addUser(User user) {
        return jdbcTemplate.update("insert into user (username,address) values (?,?);",
                user.getUsername(), user.getAddress());
    }

    /**
     * 查询方式一
     * 当类属性和数据库字段不对应时才这样使用
     * @return
     */
    public List<User> getAllUserFirst() {
        return jdbcTemplate.query("select * from user;", new RowMapper<User>() {
            @Override
            public User mapRow(ResultSet resultSet, int i) throws SQLException {
                User user = new User();
                int id = resultSet.getInt("id");
                String address = resultSet.getString("address");
                String username = resultSet.getString("username");
                user.setId(id);
                user.setUsername(username);
                user.setAddress(address);
                return user;
            }
        });
    }

    /**
     * 查询方式二
     * 当类属性和数据库字段对应时就这样使用啦，比上面的简洁很多
     */
    public List<User> getAllUserSecond() {
        return jdbcTemplate.query("select * from user;", new BeanPropertyRowMapper<>(User.class));
    }
}
```
这里需要记一下，jdbc不论新增，修改，删除都是使用`update`方法。而查询则是使用`query`。
如果数据库字段和实体类属性不一致时，则需要使用上面代码中的`查询方式一`
如果数据库字段和实体类属性全都一致时，则可以使用上面代码中的`查询方式二`，简单快捷。

## 测试
整理完后当然是少补了测试的啦，测试类如下：
```java
@SpringBootTest
class JdbcApplicationTests {

    @Autowired
    UserService userService;

    @Test
    public void addUser() {
        User user = new User();
        user.setUsername("johnson2");
        user.setAddress("colablog.cn");
        userService.addUser(user);
    }

    public void queryUsers() {
        List<User> allUserFirst = userService.getAllUserFirst();
        System.out.println(allUserFirst);
    }
}
```

<br>

# 整合MyBatis
当下最流行的持久层框架MyBatis，天天SSM，听到耳朵都起茧子了。整合MyBatis可能是使用到最多的，整合如下：
## maven依赖
版本的话可以查看[maven仓库](https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter)
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
```

## 扫描Mapper
需要提供mapper路径给SpringBoot进行扫描,我的包扫描路径为`cn.colablog.mybatis.mapper`
方式一：自己添加一个配置项
```java
@Configuration
@MapperScan(basePackages = "cn.colablog.mybatis.mapper")
public class MyBatisConfig {
}
```
方式二：直接在Application上配置
```java
@SpringBootApplication
@MapperScan(basePackages = "cn.colablog.mybatis.mapper")
public class MybatisApplication {
    public static void main(String[] args) {
        SpringApplication.run(MybatisApplication.class, args);
    }
}
```

## Mapper映射
### UserMapper接口
在Mapper包`cn.colablog.mybatis.mapper`目录下添加UserMapper接口
```java
@Mapper
public interface UserMapper {
    List<User> getAllUser();
}
```

### UserMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.colablog.mybatis.mapper.UserMapper">
    <select id="getAllUser" resultType="com.colablog.mybatis.bean.User">
        select * from user
    </select>
</mapper>
```

**存放方式有三种:**
**方式一（默认）**
SpringBoot默认找Mapper.xml是在resources目录下，例如映射`User`类的路径在java目录下的`cn.colablog.mybatis.mapper`。
那么`UserMapper.xml`就需要放在resources目录下的`cn.colablog.mybatis.mapper`.
注意：如果你使用的是IDEA开发工具,resource下添加目录不能这样添加：
![](https://img2018.cnblogs.com/blog/1377046/201911/1377046-20191117120034029-711320206.png)
这样添加IDEA只会帮你添加一个名为`cn.colablog.mybatis.mapper`的目录，所以你需要逐个目录依次添加，存放位置如下：
![](https://img2018.cnblogs.com/blog/1377046/201911/1377046-20191118102009060-1761165564.png)

**方式二**
在properties文件中进行配置存放路径：
```xml
mybatis.mapper-locations=classpath:/mapper/*.xml
```
存放位置如下：
![](https://img2018.cnblogs.com/blog/1377046/201911/1377046-20191118102030607-89105028.png)


**方式三**
在pom.xml中配置resource需要加载java目录下的xml文件：
```xml
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
        ...
    </build>
```
这样你可以和UserMapper接口存放在同一个目录下，存放位置如下：
![](https://img2018.cnblogs.com/blog/1377046/201911/1377046-20191118101908442-1626034446.png)

文章到这里就结束啦！接下来我会继续编写关于SpringBoot的文章，有兴趣的话可以看看我的前两篇关于`SpringBoot Web篇`文章哦。
感谢各位的阅读，文章若有不足之处或更好的建议，请在下方留言，Thanks♪(･ω･)ﾉ。
