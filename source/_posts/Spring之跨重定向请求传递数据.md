---
title: Spring之跨重定向请求传递数据
date: 2019-12-09 21:10:39
tags: 
 - spring boot
 - Java
categories:
 - Java
---

# 摘要
在开发场景中，大部分数据都是使用请求转发（forward）进行传递，而使用重定向（redirect）传递数据可能比较少。  
那么问题来了：请求中的数据生命周期存活时间只在一个请求转发（request）中，当这个请求结束后，那么请求中所带的数据也会随着这个请求一起拜拜了。而重定向会向服务器发起两个请求，所以第一个请求的数据不就到不了第二个请求了吗？如图：

![](https://img2018.cnblogs.com/blog/1377046/201912/1377046-20191213230926905-1688244437.png)

如果我们想传递的数据在第二个请求中有效，那么怎么办呢？
有以下两种方法可以解决：
> url路径传递  
> 使用flash属性

---

# url路径传递
url 路径传递是比较简单的一种选择方式，因为重定向和请求转发不同，所以在重定向时必须要前面加上`redirect:`(不加的话默认就为请求转发)：
下面为重定向到`colablog`路径下，传递`{username}`参数：如下：
```java
	// 如 "redirect:/colablog/johnson"
	return "redirect:/colablog/{username}" 
```

还有一种方式是使用模板方式来定义重定向的URL，如：
```java
    @GetMapping("/red")
    public String redirect(Model model) {
        User user = ...;
        model.addAttribute("username", user.getUsername());
        return "redirect:/colablog/{username}";
    }
```
若 `user.getUsername()` 为 johnson，那么重定向的url将会变成`redirect:/colablog/johnson`。

---

# 使用flash属性
可以发现，使用url传递的都是一些比较简单的数据，当我们需要传递对象时，可要怎么办呢？Spring提供了数据发送为flash功能，flash属性会一直携带这些数据直到下一次请求，然后才会消失。提供实现的方法为`RedirectAttributes`的`addFlashAttribute`方法。如下：
```java
    @GetMapping("/test")
    public String test(RedirectAttributes attributes){
        User user = ...;
        attributes.addFlashAttribute("user", user);
        return "redirect:/colablog";
    }
```
取出数据还是老样子，像请求转发(forward)那样获取数据。
```java
    @GetMapping("/colablog")
    public String colaBlog(Model model) {
        User user = model.getAttribute("user");
        return "success";
    }
```

`RedirectAttributes`有`Model`类的所有方法，因为`RedirectAttributes`是`Model`的扩展类。
```java
public interface RedirectAttributes extends Model {}
```
至于为什么使用flash属性会携带到下一次请求中，然后才会消失呢？因为该flash属性的数据会存放到会话当中，在重定向后，存在会话中的flash属性会被取出，从会话数据转移到模型数据之中。如下图：
![](https://img2018.cnblogs.com/blog/1377046/201912/1377046-20191213214739765-1313490863.png)

好了，文章到这里就结束了，不知道各位小伙伴看懂了没。若还有问题可在下方留言，Thanks♪(･ω･)ﾉ

参考文献：[《Spring实战 第4版》](https://book.douban.com/subject/26767354/)