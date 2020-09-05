---
title: GitPage部署自己的项目
date: 2019-08-14 21:06:06
tags:
 - GitPages
categories:
 - Git
---

# 前言
该文章主要为了记录我如何在GitPages上面部署博客网站，这里的话，码云上面也有相同的功能。
若有小伙伴担心GitHub担心把中国的访问也禁了的话（大概不会吧），可以在码云上面部署。流程应该是差不多的。
因为我使用的域名是.cn后缀，所以部署到GitHub上面就不用备案了。码云是国内的，应该要备案了，这个就看各位小伙伴的选择了。
可以看看我的网站：
> https://colablog.cn/

  

<br>

# 开始
## 第一步，安装工具
我们需要创建一个空的项目，怎么创建呢？这里我是使用Hexo的博客框架，
他会使用Markdown引擎快速渲染出静态页面。
安装hexo的前提是需要安装好下面的应用程序：

> Node.js (Should be at least nodejs 6.9)  
> Git

然后使用npm安装Hexo。(建议去配置个淘宝的cnpm镜像，快贼多)
```text
$ npm install -g hexo-cli
```
  
<br>

## 第二步，hexo创建项目
我们需要使用hexo建立一个空的项目,这里的项目名为blog。
```text
$ hexo init blog
$ cd blog
$ npm install
```

为了在可以在本地调试效果，我们需要安装hexo-server，就是Hexo的服务器
```text
$ npm install hexo-server --save
```

然后启动hexo-server，访问的网址的`localhost:4000`
```text
$ hexo server
```

启动后应该可以看见下面的界面
![hexo-server页面](http://qiniuyun.colablog.cn/hexo-server%E5%90%AF%E5%8A%A8%E9%A1%B5%E9%9D%A2.jpg)

新建名为test的文章测试一下，创建好后在`locaohost:4000`可以看到新的文章哦。
```text
$ hexo new post test //全写
$ hexo new test //简写，默认为post（文章）
```
到此为止你已经可以在本地建好博客网站啦。  

<br>

## 第三步 使用NexT主题(可跳过)
hexo也有推荐使用的主题列表，入口在这：
> https://hexo.io/themes/

不过我没有去看这些主题，我是使用了NexT的主题，入口在这：
> http://theme-next.iissnan.com/theme-settings.html#author-sites

首先我们克隆最新的NexT版本，
```text
$ cd <你的项目目录>
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

然后在hexo的配置文件(_config.yml)文件里面，找到`theme`字段，修改如下：
```text
theme: next
```

个人觉得next默认的主题样式还是比较丑的，我们可以在next主题下看到还有另外三种样式，搜索关键字
`Schemes`可以看到如下主题，我使用的第三个`Pisces`
```text
# Schemes
#scheme: Muse
#scheme: Mist
scheme: Pisces
#scheme: Gemini
```
主题如下：
![next主题](http://qiniuyun.colablog.cn/next%E4%B8%BB%E9%A2%98.jpg)
  
看着是不是怪丑的，特别是第二篇文章，怎么会展示这么多呢？其实可以调整的，反正我是找了好久，在next主题下，
搜索关键字`auto_excerpt`,修改如下：
```text
auto_excerpt:
  enable: true //开启该功能
  length: 150 //首页展示的字数限制
```

到此为止我们已经可以使用NexT主题啦。更详细的配置就进去官网看看吧（上面有）。

  

<br>

## 第四步 部署到GitHub
首先，我们要在GitHub上面创建一个仓库，这里我叫做blog吧。
然后我们需要把我们本地的blog项目初始化一下。
```text
$ cd <你的本地项目目录>
$ git init
$ git add -A //把全部都添加进去吧，也没啥
$ git commit -m "首次提交"
$ git remote add origin <你自己的仓库路径,例：https://github.com/xxx/blog.git>
$ git push -u origin master
```

然后我们进入blog仓库的setting中，然后往下拉看到`GitHub Pages`
![GitHub Pages](http://qiniuyun.colablog.cn/GitPage-1.jpg)

修改完后页面会自己刷新，然后重新回到`GitHub Pages`这部分。你会看到他给了你一个网址，没错！就是这个网址，
你打开试试！！试试就试试，404...。  
你先记住这个网址，咱们先把这个网址叫做`博客网址`吧。

其实部署到GitPages上面的话，hexo还是要做一些设置的，不然他怎么知道你要部署到那个地方去哦。设置完后以后部署文章会很简单的：  
设置你项目的root路径，在hexo配置文件(_config.yml)中，搜索关键字`root`, 改为你的仓库名称,如下：
```text
# URL
root: /blog
```
在hexo的配置文件(_config.yml)中，搜索关键字`deploy`(其实就在最下面)，设置如下：
```text
deploy:
  type: git
  repo: <你的仓库地址> //https://github.com/xxx/blog.git
  branch: master
```
安装hexo-deployer-git，
```text
$ cd <你的项目目录>
$ npm install hexo-deployer-git --save
```
然后你再执行下面这条命令就OK了！
```text
$ hexo generate --deploy //全写
$ hexo g -d //缩写
```
赶紧打开上面说的`博客网址`看看，是不是404！，没错！
等一会吧，GitHub还没缓过来呢，执行完命令之后大概差不多一分钟之后刷新一下，
你就可以看到你梦寐以求的页面了。  
以后咱们创建文章就很简单了，新建并且编写好文章之后，执行使用部署到服务器的命令就Ok了。操作如下：
```text
$ hexo new <文章名> //新建文章
$ hexo g -d //部署到GitHub，你就可以看到的新文章啦！
```


毕竟第一次难免是比较困难。嗯，没错，我说的是部署GitPages。
如果你也是跟着我这篇文章一步一步走的话，应该是没什么毛病的，因为我是自己重新部署一个项目的，
然后一步一步的把步骤记录下来的。如果有什么问题的话，可以留言一下，或致邮箱821312534@qq.com。谢啦。