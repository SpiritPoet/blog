---
title: Hexo博客搭建
date: 2019-05-30 14:24:09
categories:
    - 工欲善其事必先利其器
tags:
    - 利器-博客
---

![logo](banner.jpg)
<!-- more -->
## 准备工作 
>1.注册一个[GitHub](https://github.com/)帐号
>2.安装[node.js](https://nodejs.org/zh-cn/download/)
>3.安装[Git](https://git-scm.com/)客户端

## Hexo安装及搭建
安装及启动搭建---参考官网[https://hexo.io/zh-cn/](https://hexo.io/zh-cn/)

## 结合GitHub
**1.创建页面仓库**
* step 1
![新建仓库](1.jpg)
* step 2
![命名](2.jpg)

**2.生成SSH密匙(运用git工具)**

    ssh-keygen -t rsa -C "你的邮箱地址" 

>(>按3个回车，密码为空。)

**3.在Github上添加SSH**
>生成密匙后,在 /Users/用户名/.ssh/下,会得到两个文件 **id_rsa** 和**id_rsa.pub**,打开**id_rsa.pub**文件全部复制(CTRL +A ,CTRL + C) 进入Github的settings 粘贴.

* step 1
![设置ssh](3.jpg)
* step 2
![设置ssh](4.jpg)
* step 3
![设置ssh](5.jpg)
* step 4
![设置ssh](6.jpg)

>密匙的作用:就是用来向git推从信息的. 它是一种安全的传输模式 发布的时候要用到它

**4.配置hexo**

>现在我们打开我们初始化的blog文件夹,这个文件加的目录结构是这样的
>
>├── .deploy #需要部署的文件
>├── node_modules #Hexo插件
>├── public #生成的静态网页文件
>├── scaffolds #模板
>├── source #博客正文和其他源文件，404、favicon、CNAME 都应该放在这里
>| ├── _drafts #草稿
>| └── _posts #文章
>├── themes #主题
>├── _config.yml #全局配置文件
>└── package.json


>根据说明配置[主题、评论、打赏、RSS等功能](https://www.jianshu.com/p/5973c05d7100)



**小技巧**

```
常用命令
hexo help #查看帮助
hexo init #初始化一个目录
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成网页，可以在 public 目录查看整个网站的文件
hexo server #本地预览，'Ctrl+C'关闭
hexo deploy #部署.deploy目录
hexo clean #清除缓存，强烈建议每次执行命令前先清理缓存，每次部署前先删除 .deploy 文件夹
```
```
简写
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```

**扩展阅读**
>1.换电脑了博客怎么办---->[Hexo备份github](http://blog.csdn.net/u012195214/article/details/72721065)
>2.嫌弃github域名--->[hexo+github搭建的博客绑定域名](https://www.jianshu.com/p/cea41e5c9b2a)

**注:此为我的学习笔记,有些内容借鉴了其他文章**

