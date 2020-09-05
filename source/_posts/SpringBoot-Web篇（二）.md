---
title: SpringBoot-Web篇（二）
date: 2019-11-18 21:39:49
tags: 
 - spring boot
 - Java
categories:
 - Java
---

# 摘要
继上一篇 [SpringBoot Web篇（一）](https://www.cnblogs.com/Johnson-lin/p/11846337.html)

<br>

# 文件上传
当我们服务器需要接收用户上传的文件时，就需要使用`MultipartFile`作为参数接收文件。如下：
```java
    @PostMapping("/upload")
    public String uploadFile(MultipartFile file, HttpServletRequest request) {
        String format = sdf.format(new Date()); //格式化当前日期
        //文件保存的目录, 根据自己需求定义
        String filePath = request.getServletContext().getRealPath("/img") + format;
        File folder = new File(filePath);

        if (!folder.exists()) {
            folder.mkdirs(); // 注意是mkdirs 不是mkdir,需要递归生成目录
        }

        // 获取上传的文件名
        String oldName = file.getOriginalFilename();
        // 使用UUID和文件后缀组成新的文件名
        String newName = UUID.randomUUID().toString() + oldName.substring(oldName.lastIndexOf("."));

        try {
            // 保存文件
            file.transferTo(new File(folder, newName));
            // 返回保存的路径
            String url = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + "/img" + format + newName;
            return url;
        } catch (IOException e) {
            e.printStackTrace();
        }

        return "error";
    }
```
当上传多文件时，使用`MultipartFile[]`进行接收，或是多个`MultipartFile`, 这个需要根据表单上传的方式决定。
如果表单上传方式如下：则使用`MultipartFile[]`：
```html
   <input type="file" name="files" multiple>
```
如果表单上传方式如下：则使用多个`MultipartFile`：
```html
   <input type="file" name="file1">
   <input type="file" name="file2">
```
<br>

# 路径映射
当我们直接可以访问某个动态页面而不需要经过控制器时，我们可以如下设置：
```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/java")
                .setViewName("hello");
    }
}
```
输入`localhost:8080/java`时，会直接访问到hello页面

<br>

# 类型转换器
当用户输入 2019-11-14 这样的数据时，如何在后台转换为Date类型呢？如下：
```java
    @GetMapping("/hello")
    public void hello(Date birth){
        System.out.println(birth);
    }
```
这时就可以用到Spring的转换器`Converter`,代码实现如下：
```java
@Component
public class DateConverter implements Converter<String, Date> {

    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

    @Override
    public Date convert(String s) {
        if (s != null && "".equals(s)) {
            try {
                return sdf.parse(s);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
        return null;
    }
}
```
访问`localhost:8080/hello?birth=2019-11-14`的url时，会自动帮你把2019-11-14转为Date类型。

<br>

# AOP
Spring AOP面向切面编程，可以切入到业务逻辑中做统一处理。例如事务、做日志、权限验证、请求...，
pom.xml下添加如下依赖：
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

AOP切入类如下, 需要在切入类添加`@Aspect`注解，详细的说明请在代码中查看
```java
@Component
@Aspect
public class AopComponent {

    /**
     * 设置切入点
     * org.java代表目录，
     * 第一个 * 代表该目录下的所有类
     * 第二个 * 代表类下的所有方法
     * (..) 代表方法下所有的参数
     * 得出：切入org.java目录下的所有方法
     */
    @Pointcut("execution(* org.java.*.*(..))")
    public void pcl() {
    }

    /**
     * 进入方法前调用
     */
    @Before(value = "pcl()")
    public void before(JoinPoint jp) {
        String name = jp.getSignature().getName(); //方法名
        System.out.println("before--" + name);
    }

    /**
     * 方法结束后调用
     */
    @After(value = "pcl()")
    public void after(JoinPoint jp) {
        String name = jp.getSignature().getName();
        System.out.println("after--" + name);
    }

    /**
     * 有返回值才调用，且在@After前调用，并获取到result（返回值）
     */
    @AfterReturning(value = "pcl()", returning = "result")
    public void afterReturning(JoinPoint jp, Object result) {
        String name = jp.getSignature().getName();
        System.out.println("afterReturning--" + name + "----" + result);
    }

    /**
     * 抛异常时调用
     * 不会调用@AfterReturning和不会调用around（因为没有返回值）
     */
    @AfterThrowing(value = "pcl()", throwing = "e")
    public void afterThrowing(JoinPoint jp, Exception e) {
        String name = jp.getSignature().getName();
        System.out.println("afterThrowing--" + name + "----" + e.getMessage());
    }

    /**
     * 有返回值才调用
     * 对返回的数据进行处理，例如response设置统一返回格式
     * 在@AfterReturning前调用
     */
    @Around("pcl()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        Object proceed = joinPoint.proceed();
        System.out.println("around--" + proceed.toString());
        return proceed;
    }
}
```
总的来说，内容如下：
> @Aspect: 标明该类为切面类
> @Pointcut: 设置切入点
> @Before: 进入方法前调用
> @After: 方法结束后调用
> @AfterReturning: 有返回值才调用，且在@After前调用，并获取到result（返回值）
> @AfterThrowing: 抛异常时调用，不会调用@AfterReturning和不会调用around（因为没有返回值）
> @Around: 有返回值才调用，对返回数据进行处理，在@AfterReturning前调用

<br>

# 浏览器的标签图标
就是修改浏览器上面的这个图标：
![](https://img2018.cnblogs.com/blog/1377046/201911/1377046-20191116160240301-948192476.png)

把名为favicon.ico图标放在下面这两个目录都可以：
> resources.static
> resources

我是使用的时Google浏览器，更换后没有即刻生效
重新打开浏览器就好了，应该时浏览器访问后就会把favicon.ico缓存起来。

<br>

# 除去自动化配置
一般来说使用SpringBoot都不会用到，使用SpringBoot不就是贪图他的自动化配置吗？不过他还是提供了除去自动化配置的功能：
## 方式一 Application
在你的启动文件xxxApplication中, 例如除去ErrorMvcAutoConfiguration：
```java
@SpringBootApplication(exclude = ErrorMvcAutoConfiguration.class)
...
```

## 方式二 properties或yml
```properties
spring.autoconfigure.exclude=...ErrorMvcAutoConfiguration
```

<br>

若文章有错误或疑问，可在下方评论，Thanks♪(･ω･)ﾉ。