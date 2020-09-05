---
title: 优雅的使用BeanUtils对List集合的操作
date: 2019-12-30 22:06:02
tags:
 - Java
 - 工具类
categories:
 - Java
 
---

# 摘要
我们在Entity、Bo、Vo层数据间可能经常转换数据，Entity对应的是持久层数据结构（一般是数据库表的映射模型）、Bo对应的是业务层操作的数据结构、Vo就是Controller和客户端交互的数据结构。在这些数据结构之间很大一部分属性都可能会相同，我们在使用的时候会不断的重新赋值。  
如：客户端传输管理员信息的到Web层，我们会使用AdminVo接收，但是到了Service层时，我就需要使用AdminBo，这时候就需要把AdminVo实例的属性一个一个赋值到AdminBo实例中。

# BeanUtils
Spring 提供了 **org.springframework.beans.BeanUtils** 类进行快速赋值，如：
AdminEntity类
```java
public class AdminEntity{
    private Integer id;
    
    private String password;

    private String username;

    private String userImg;
    .... //一些 Set Get方法
}
```

AdminVo类，因为是和客户端打交道的，所以password属性就不适合在这里了

```java
public class AdminVo{
    private Integer id;

    private String username;

    private String userImg;
    .... //一些 Set Get方法
}
```

假如我们需要把AdminEntity实例属性值赋值到AdminVo实例中（暂时忽略Bo层吧）
```java
    AdminEntity entity = ...;
    AdminVo vo = new AdminEntity();
    // org.springframework.beans.BeanUtils
    BeanUtils.copyProperties(entity, vo); // 赋值
```

那么这样AdminVo实例中的属性值就和AdminEntity实例中的属性值一致了。
但是如果我们是一个集合的时候就不能这样直接赋值了。如：
```java
        List<Admin> adminList = ...;
        List<AdminVo> adminVoList = new ArrayList<>(adminList.size());
        BeanUtils.copyProperties(adminList, adminVoList); // 赋值失败
```
这样直接赋值是不可取的，由方法名（copyProperties）可知，只会复制他们的属性值，那么上述的`adminList`属性和`adminVoList`的属性是没有半毛钱关系的。那么怎么解决了？

## 方式一（暴力解决，不推荐）
代码如下：
```java
        List<Admin> adminList = ...;
        List<AdminVo> adminVoList = new ArrayList<>(adminList.size());
        for (Admin admin : adminList) {
            AdminVo adminVo = new AdminVo();
            BeanUtils.copyProperties(admin, adminVo);
            adminVoList.add(adminVo);
        }
	return adminVoList;
```
虽然`for`循环可以解决，但是一点都不优雅，这样的代码也会陆续增多，代码多了，就会看出来了重复的地方，没错！就是`for`循环赋值的地方，完全可以优化掉（代码写多了一眼就看出来了）。那么请看优雅的方式二

## 方式二 （优雅、推荐）
这也是我第一次写泛型的代码，可能有待提高，如下：
ColaBeanUtils类（Cola是我家的狗狗名，哈哈）
```java
import org.springframework.beans.BeanUtils;

public class ColaBeanUtils extends BeanUtils {

    public static <S, T> List<T> copyListProperties(List<S> sources, Supplier<T> target) {
        return copyListProperties(sources, target, null);
    }

    /**
     * @author Johnson
     * 使用场景：Entity、Bo、Vo层数据的复制，因为BeanUtils.copyProperties只能给目标对象的属性赋值，却不能在List集合下循环赋值，因此添加该方法
     * 如：List<AdminEntity> 赋值到 List<AdminVo> ，List<AdminVo>中的 AdminVo 属性都会被赋予到值
     * S: 数据源类 ，T: 目标类::new(eg: AdminVo::new)
     */
    public static <S, T> List<T> copyListProperties(List<S> sources, Supplier<T> target, ColaBeanUtilsCallBack<S, T> callBack) {
        List<T> list = new ArrayList<>(sources.size());
        for (S source : sources) {
            T t = target.get();
            copyProperties(source, t);
            list.add(t);
            if (callBack != null) {
 		// 回调
                callBack.callBack(source, t);
            }
        }
        return list;
    }
```

ColaBeanUtilsCallBack接口，使用java8的lambda表达式注解：
```java
@FunctionalInterface
public interface ColaBeanUtilsCallBack<S, T> {

    void callBack(S t, T s);
}
```

使用方式如下：
```java
	List<AdminEntity> adminList = ...;
        return ColaBeanUtils.copyListProperties(adminList, AdminVo::new);
```
如果需要在循环中做处理(回调)，那么可使用lambda表达式：
```java
        List<Article> adminEntityList = articleMapper.getAllArticle();
        List<ArticleVo> articleVoList = ColaBeanUtils.copyListProperties(adminEntityList , ArticleVo::new, (articleEntity, articleVo) -> {
            // 回调处理
        });
        return articleVoList;
```
简直不要太简单了！！！  

# 总结
`AdminVo::new`配合`Supplier<T> target`这里我完全是参考《Java核心技术第十版 卷一》的第八章泛型，因为之前看过有点印象，没想到今天用到了。`@FunctionalInterface`这个是java8的`lambda`表达式的注解类，参考`java.util.function.Consumer`接口。没想到懵懵懂懂的，就把之前看过的知识写出来了，惊呆了，哈哈。
代码如果雷同，纯属巧合。若泛型的代码还有可以改进的地方，可在下方留言，非常感谢，Thanks♪(･ω･)ﾉ。