---
title: Java核心技术第八章-泛型
date: 2019-12-04 11:01:42
tags:
 - Java核心技术
categories:
 - Java
---

# 摘要
本文根据[《Java核心技术 卷一》](https://book.douban.com/subject/26880667/)一书的第八章总结而成，部分文章摘抄书内，作为个人笔记。
文章不会过于深入，望读者参考便好。

# 为什么要使用泛型程序设计
泛型程序设计（Generic programming) 意味着编写的代码可以被很多不同类型的对象所重用。

## 类型参数的好处
在没有泛型类之前，ArrayList类只维护一个Object引用的数组：
```java
public class ArrayList {
   private Object[] elementData; // 用于存放数据
   public Object get(int i) { . . . }
   public void add(Object o) { . . . }
	 ...
}
```

**问题** 
> 1.获取一个值时必须进行强制类型转换  
> 2.这里没有错误检査。可以向数组列表中添加任何类的对象，如果数组的类型不一致，将 get 的结果进行强制强制类型，就会错误。

泛型提供了一个更好的解决方案： 类型参数：
```java
ArrayList<String> array = new ArrayList<String>():
```
利用类型参数的信息，我们就可以在添加数据的时候保持类型统一，调用get方法时候也不需要进行强制类型转换，因为我们在初始化的时候就定义了类型，编译器识别返回值的类型就会帮我们转换该类型。

# 定义一个简单泛型类
```java
public class Pair<T> {
    private T num;

    public Pair(T num) { this.num = num; }
    public T getNum() { return num; }
    public void setNum(T num) { this.num = num; }
}
```
我们可以看到，在Pair类名后面添加了一个<T>，这个是泛型类的类型变量，而且还可以有多个类型变量，如
```java
public class Pair<T,U> {
    ...
}
```
如果我们实例化Pair类，例如：
```java
new Pair<String>;
```
那么我们就可以把上述的Pair类想象成如下：
```java
public class Pair<String> {
    private String num;

    public Pair(String num) { this.num = num; }
    public String getNum() { return num; }
    public void setNum(String num) { this.num = num; }
}
```
是不是很简单呢？在Java库中，使用变量E表示集合的元素类型，K和V分别表示表的关键字与值的类型，T、U、S表示任意类型。

# 泛型方法
定义一个带有类型参数的方法
```java
    public static <T> T getMiddle(T... a) {
        return a[a.length / 2];
    }
```
可以看到类型变量(< T >)放在修饰符( public static )的后面，返回类型（T）的前面。泛型方法可以定义在普通类或泛型类中。

# 类型变量的限定
如果我们需要对类型变量加以约束，例如：传入的变量必须实现Comparable接口，因为需要该变量调用compareTo的方法。这样我们就可以使用`extends`关键字对变量进行限定。
```java
    public <T extends Comparable> T max(T a) {
        a.compareTo(...);
        ...
    }
```
无论变量需要限定为继承某个类或者实现某个接口，都是使用`extends`关键字进行限定。

# 泛型代码和虚拟机
## 类型擦除
无论我们在代码中怎么定义一个泛型类、泛型方法，都提供了一个相应的原始类型。原始类型的名字就是删去类型参数后的泛型类姓名。如 `<T>` 的原始类型为 `Object`，`<T extends MyClass>`的原始类型为`MyClass`。
代码就像下面这样：
```java
public class Pair<T> {
    private T property;

    public Pair(T property) {
        this.property = property;
    }
}
```
类型擦除后：
```java
public class Pair<Object> {
    private Object property;

    public Pair(Object property) {
        this.property = property;
    }
}
```

## 翻译泛型表达式
如果擦除返回类型，编译器会插入强制类型转换，就像下面这样：
```java
Pair<Employee> buddies = . .
Employee buddy = buddies.getFirst()；
```
擦除getFirst的返回类型后将返回Object类型，但是编译器将自动帮我们强制类型转换为Employee。
所以：编译器把这个方法执行操作分为两条指令：

> 对原始方法Pair.getFirst的调用
> 将返回的Object类型强制转换为Employee类型

小节总结：
> 虚拟机中没有泛型，只有普通的类和方法
> 所有的类型参数都用他们的限定类型替换
> 为保持类型安全性，必要时插入强制类型转换
> 桥方法被合成来保持多态（本文没有讲到，不过桥方法可以忽略，Java编写不规范才会有桥方法生成）

# 约束与局限性
## 不能用基本类型实例化类型参数
不可以用八大基本数据类型去实例化类型参数，你也没见过`ArrayList<int>`这样的代码吧。只有`ArrayList<Integer>`。原因是因为基本数据类型是不属于Object的。所以只能用他们的包装类型替换。

## 运行时类型查询只适用于原始类型
所有的类型查询只产生原始类型，因为在虚拟机没有所谓的泛型类型。
例如：
```java
	Pair<String> pair = new Pair<String>("johnson");
	if (pair instanceof Pair<String>) {} // Error
	if (pair instanceof Pair<T>) {} // Error
	if (pair instanceof Pair) {} // Pass
```

## 不能创建参数化类型的数组
```java
    Pair<String>[] pairs = new Pair<String>[10]; //error
```
为什么不能这样定义呢？因`pairs`的类型是`Pair[]`，可以转换为`Object[]`，如果试图存储其他类型的元素，就会抛出ArrayStoreException异常，
```java
	pairs[0] = 10L; //抛异常
```
总之一句话，不严谨。所以不能创建参数化类型的数组。

## 不能实例化类型变量
不能使用 new T(...)、new T[...] 或 T.class。因为类型擦除后，T将变成Object，而且我们肯定不是希望实例化Object。
不过在Java8之后，我们可以使用`Supplier<T>`接口实现，这是一个函数式接口，表示一个无参数而且有返回类型为T的函数：
```java
public class Pair<T> {
    private T first;
    private T second;

    private Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }

    public static <T> Pair<T> makePair(Supplier<T> constr) {
        return new Pair<>(constr.get(), constr.get());
    }
    
	public static void main(String[] args) {
        Pair<String> pair = Pair.makePair(String::new);
    }
}
```

## 泛型类中的静态上下文中类型变量无效
不能在静态域或方法中引用类型变量。例如：
```java
public class Pair<T> {
    private static T instance; //Error

    public static T getInstance() { //Error
        if (instance == null) 
            instance = new Pair<T>();
        return instance;
    }
}
```
因为类中的类型变量`（<T>）`是在对象中的作用域有效，而不是在类中的作用域有效。如果要使用泛型方法，可以参照文章上面的**泛型方法**哦~

## 不能抛出或捕获泛型类的实例
即不能抛出也不能捕获泛型类的对象，甚至扩展Throwable都是不合法的：
```java
public class Pair<String> extend Exceotion {} //Error
```
```java
public static <T extends Throwable> void doWork(Class<T> t) {
	try {
	 	...
	} catch (T e) { //Error
		...
	}
}
```
但是在抛出异常后对异常使用类型变量是允许的（个人感觉没见过这样的代码）。
```java
public static <T extends Throwable> void doWork(Class<T> t) throw T { //Pass
	try {
	 	...
	} catch (Exception e) {
		throw e;
	}
}
```

# 泛型类型的继承规则
如`Manager`类继承`Employee`类。但是`Pair<Employee>`和`Pair<Manager>`是没有关联的。就像下面的代码，就会提示报错，传递失败：
```java
        Pair<Manager> managerPair = new Pair<Manager>();
        Pair<Employee> employeePair = managerPair; //Error
```

# 通配符类型
## 通配符概念
通配符类型中，允许类型参数变化，使用 `?` 标识通配符类型：
```java
Pair<? extends Employee>
```
若Pair类如下
```java
public class Pair<T> {
    private T object;

    public void setObject(T object) {
        this.object = object;
    }
}
```
那么使用通配符可以解决**泛型类型的继承规则**问题，如：
```java
        Pair<Manager> managerPair = new Pair<Manager>();
        Pair<? extends Employee> employeePair = managerPair; //Pass
        Manager manager = new Manager();
        employeePair.setObject(manager); //Error
```
使用<? extends Employee>，编译器只知道employeePair 是Employee的子类，但是不清楚具体是什么子类，所以最后一步`employeePair.setObject(manager)`不能执行。

## 通配符的超类型限定
通配符还有一个附加的能力，就是可以指定一个**超类型限定**，如：
```java
public class Pair<T> [
	...
	public static void salary(Pair<? super Manager> result) {
        //...
    }
}
```
`<? super Manager>`这个通配符为Manager的所有超类型（包含Manger），例如：
```java
        Pair<Manager> managerPair = new Pair<Manager>();
        Pair<Employee> employeePair = new Pair<Employee>();

        Pair.salary(managerPair); //Pass
        Pair.salary(employeePair); //Pass

		// 假如 ManagerChild为Manager子类
        Pair<ManagerChild> managerChildPair = new Pair<ManagerChild>();
        Pair.salary(managerChildPair); //Error
```

## 无限定通配符
无限定通配符，如：`Pair<?>`，当我们不需要理会他的实际类型时候，就可以使用无限定通配符，上代码：
```java
    public static boolean hasNull(Pair<?> pair) {
        return pair.getObject() == null;
    }
```

说实话，通配符搞得我头昏脑胀的，反复不断地看文章，才开始慢慢看懂，我太难了。。。
<br>
文章到这里就结束啦，不知道各位小伙伴看懂了没，没看懂的话可能是我的功底和文章写作能力还有待提高，小伙伴们也可以去看一下[《Java核心技术 卷一》](https://book.douban.com/subject/26880667/)这本书呢，感觉还是挺不错的。最近把这本书捡起来看也是发现基础是非常重要的，先把基础沉淀好了，再学习其他的技术点也会更容易入手，也会知其然知其所然。最近非常喜欢的一句话，送给大家：“万丈高楼平地起，勿在浮沙筑高台”。
> 个人博客网址： https://colablog.cn/

如果我的文章帮助到您，可以关注我的微信公众号，第一时间分享文章给您  

![微信公众号](https://img2018.cnblogs.com/blog/1377046/201912/1377046-20191204114856645-600707643.jpg)
