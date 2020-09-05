---
title: Java移位运算符
date: 2019-10-29 07:23:49
tags:
 - Java
categories:
 - Java
---

# 简述
Java有三种移位运算符，分别为：
1. 左移运算符 <<
2. 右移运算符 >>
3. 无符号右移运算符 >>>

首先，移位运算符根据名字可知是使用二进制进行运算的。在Integer.java中，我们可以看到有两个静态常量，`MIN_VALUE` 和 `MAX_VALUE`，这两个常量控制了Integer的最小值和最大值，如下：
```java
    /**
     * A constant holding the minimum value an {@code int} can
     * have, -2<sup>31</sup>.
     */
    @Native public static final int   MIN_VALUE = 0x80000000;

    /**
     * A constant holding the maximum value an {@code int} can
     * have, 2<sup>31</sup>-1.
     */
    @Native public static final int   MAX_VALUE = 0x7fffffff;
```
注释上说明这两个值得范围：
> MIN_VALUE（最小值） = -2^31 = -2,147,483,648‬ 
> MAX_VALUE（最大值） = 2^31 = 2,147,483,647

在32位运算中，首位为1则代表负数，0则代表正数，如：  
> 1000 0000 0000 0000 0000 0000 0000 0000 负数，该值等于MIN_VALUE
> 0111 1111 1111 1111 1111 1111 1111 1111 正数，该值等于MAX_VALUE

根据上述可知，Integer是32位运算的。

<br>

# 左移运算符 <<
使用 << 时，需要在低位进行补0，例子如下：
```java
        int a = 3;
        System.out.println(Integer.toBinaryString(a));

        int b = a << 1;
        System.out.println(Integer.toBinaryString(b));
        System.out.println(b);

        System.out.println("----------------------------------------------");

        int c = -3;
        System.out.println(Integer.toBinaryString(c));

        int d = c << 1;
        System.out.println(Integer.toBinaryString(d));
        System.out.println(d);
```
输入如下：
```text
11
110
6
----------------------------------------------
11111111111111111111111111111101
11111111111111111111111111111010
-6
```
可以清楚的看到 3 << 1 时，在后面补0，得到 110 即等于6;

<br>

# 右移运算符 >>
右移运算符时，正数高位补0，负数高位补1。如：
```java
        int a = 3;
        System.out.println(Integer.toBinaryString(a));

        int b1 = a >> 1;
        System.out.println(Integer.toBinaryString(b1));
        System.out.println(b1);

        System.out.println("----------------------------------------------");

        int c = -3;
        System.out.println(Integer.toBinaryString(c));

        int d = c >> 1;
        System.out.println(Integer.toBinaryString(d));
        System.out.println(d);
```
输出如下：
```text
11
1
1
----------------------------------------------
11111111111111111111111111111101
11111111111111111111111111111110
-2
```

<br>

# 无符号右移 >>>
在正数当中，>> 和 >>> 是一样的。负数使用无符号右移时，则高位不进行补位。
```java
        int c = -3;
        System.out.println(Integer.toBinaryString(c));

        int d = c >>> 1;
        System.out.println(Integer.toBinaryString(d));
        System.out.println(d);
```
输出如下：
```text
11111111111111111111111111111101
1111111111111111111111111111110
2147483646
```

<br>

#总结
> 左移运算符 << ： 需要在低位进行补0
> 右移运算符 >> ： 正数高位补0，负数高位补1
> 无符号右移运算符 >>> ：在正数当中，>> 和 >>> 是一样的。负数使用无符号右移时，则高位不进行补位