---
title: CentOS搭建Nginx环境
date: 2019-09-02 20:40:21
tags: 
 - nginx
categories:
 - nginx
---

# 准备Nginx的依赖软件
## GCC编译器
GCC编译器和G++，用于编写Nginx HTTP模块
```text
yum install -y gcc
yum install -y gcc-c++
```

## PCRE库
函数库，支持正则表达式，如果在nginx.conf里面使用了正则表达式，那么编译Nginx时就必须引进PCRE库，用于解析HTTP模块的正则表达式,
如果你不会用到正则表达式则可以忽略。

```text
yum install -y pcre pcre-devel
```

## zlib库
用于对http包的内容做gzip格式的压缩。
```text
yum install -y zlib zlib-devel
```

## OpenSSL开发库
使用SSL协议上安全传输HTTP，就是所谓的https。
```text
yum install -y openssl openssl-devel
```

# 安装Niginx
首先当Nginx官网下载源码包，官网下载地址：http://nginx.org/en/download.html  
也可以和我一样下载1.16.1版本。
```text
cd ~ #回到家目录
wget http://nginx.org/download/nginx-1.16.1.tar.gz #下载源码包
tar -zxvf nginx-1.16.1.tar.gz
```

然后我们开始进行编译安装Nginx，进入解压后的目录后，执行以下3行命令：
```text
./configure
make
make install
```

默认情况下，Nginx会被安装到目录/usr/local/nginx中，然后我们来启动一下Nginx吧。
```
/usr/local/nginx/sbin/nginx
```

在浏览器输入你的ip地址，就能看到`Welcome to nginx!`啦！
启动好了就该关闭掉拉，毕竟是测试，快速停止服务如下：
```
usrlocal/nginx/sbin/nginx -s stop #强制退出
usrlocal/nginx/sbin/nginx -s stop #正常退出
```
强制退出这个命令一般不太建议使用，就像电脑重装系统，安装到一半来个关机然后你就爽歪歪。
建议使用正常退出。
下一篇继续讲Niginx的，如果帮到你，请关注我啦！！~