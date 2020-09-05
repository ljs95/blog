---
title: Gitment评论插件的使用
date: 2019-08-20 07:59:49
tags:
 - GitPages
categories:
 - Git
---

# 前言
继上一篇的
[GitPages部署自己的网站](https://colablog.cn/2019/08/14/GitPage%E9%83%A8%E7%BD%B2%E8%87%AA%E5%B7%B1%E7%9A%84%E9%A1%B9%E7%9B%AE/#more)

现在开始添加博客的评论插件Gitment。这里的话我是使用hexo添加gitment插件，如果不是使用hexo，请到[官网指定这里](https://imsun.net/posts/gitment-introduction/)。

# 开始
## 第一步 注册 OAuth Application
首先在[点击这里](https://github.com/settings/applications/new)注册自己OAuth Application，
此处有四个内容：
> Application name： 随意填写  
> Homepage URL: 随意填写一个url  
> Application description: 随意填写  
> Authorization callback URL: 填你的博客网址（例: https://user.github.io/blog/），如果有域名则填域名。

注册完后你会得到一个 client ID 和一个 client secret，记住这两个玩意，等会hexo配置会用到

<br>

## 第二步 安装Gitment
在你的博客下使用npm进行安装gitment
```text
$ npm install --save gitment
```

<br>

## 第三步 修改主题配置
因为这里我使用的是Next主题配置，在配置文件搜索关键字`gitment`,主要配置如下：
```text
gitment:
  enable: true //这里必须为开启
  ...
  github_user: username
  github_repo: blog_comments #新建一个存储评论的仓库，这里填写仓库名
  client_id: #第一步注册的client_id
  client_secret: #第一步注册的client_secret
  ...
```
github_user不知道是哪个？你在github页面中，点击你的头像看到第一行显示`signed in as xxx`，
就是填写这个`xxx`。github_repo这里是让你再新建一个仓库，用来存储评论的，不是当前的这个博客的仓库，
然后填上你仓库名的名字，对！就是单纯的名字，仓库名叫`blog_comments`就填`blog_comments`。

<br>

## 第四步 初始化评论插件
搞定好以上的步骤后，你就能看到博客的下方是这样的
![Gitment图片1](http://qiniuyun.colablog.cn/Gitment-1.jpg)
点击登入后，`(未开放评论)`的地方会显示一个按钮让你初始化，点击按钮然后你就可以进行评论啦！

<br>
好了,到此为止就搞定了成功接入了Gitment插件了。如果有什么问题可以留言一下咯。Thanks♪(･ω･)ﾉ。
参考我的博客
> https://colablog.cn/