---
title: FastDFS单机搭建
date: 2019-06-05 11:01:27
tags: FastDFS
---

# FastDFS 单机版搭建

#### 目标

​ 搭建可使用的单服务器版 FastDFS 文件服务器,本文对原理不做阐述，因为作者也没有对 FastDFS 深入研究。如果后续深入研究作者一定再发文更新。

#### 简单概念介绍

架构图

![架构图](1.png)

<!-- more -->
Tracker 服务

跟踪服务器，主要负责调度 storage 节点与 client 通信，在访问上起负载均衡的作用，和记录 storage 节点的运行状态，是连接 client 和 storage 节点的枢纽

Storage 服务

存储服务器，保存文件和文件的 meta data（元数据）；

Group

文件组，也称为卷。做集群时一组会有多台服务器，同组服务器上的文件是完全相同的，当文件上传到组内的一台机器后，该文件会同步到同组内的其它所有机器上，起到备份的作用；

meta data

文件相关属性，键值对（KeyValue Pair）方式，如：width=1024, height=768；

#### 环境准备

​ 1.操作系统 Centos 6.4

​ 2.包管理工具 yum (阿里源)

​ 3.依赖安装

```shell
yum -y install perl
```

```shell
yum -y install libevent
```

```
yum install gcc-c++
```

```shell
yum install -y unzip zip
```

#### 安装

​ 1.下载相关程序包

​ [FastDFS 官方 github 下载地址](https://github.com/happyfish100)，注意：要保持版本一致否则会有各种问题。这里我都是下载的 master 版本

fastdfs-master、 libfastcommon-master、fastdfs-nginx-module-master

​ 2.解压编译安装

```shell
#首先安装fast公共依赖包
unzip -o libfastcommon-master.zip -d /usr/local
cd /usr/local/libfastcommon-master
./make.sh
./make.sh install
```

```shell
#公共依赖包安装完，安装主程序包
unzip -o fastdfs-master.zip -d /usr/local
cd /usr/local/fastdfs-master
./make.sh
./make.sh install
cp -r conf/* /etc/fdfs
```

​ 安装完 libfastcommon-master 和 fastdfs-master 两个程序包，FastDFS 就已经安装完毕了。

#### 配置启动

​ 1.配置启动 trackerd

```shell
cd /etc/fdfs/
vi tracker.conf
#常用配置项
port=22122 #tracker程序的管理端口
base_path=/home/fdfs/work/trackerd #存放tracker程序的 日志和一些数据的文件路径
http.server_port=9999 #http访问tracker程序的端口
#启动
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf
#验证是否启动
ps -ef | grep tracker
```

​ 2.配置启动 storaged

```shell
cd /etc/fdfs/
vim storage.conf
#常用配置项
port=23000 #storage程序的管理端口
base_path=/home/fdfs/work/storage  #存放文件和日志的文件路径
tracker_server=本地IP:22122 #指定tracker服务
store_path0=/home/fdfs/work/storage #多仓库配置的时候设置多个(我们单机采用默认的和base_path相同)
http.server_port=9998 #http访问tracker程序的端口
#启动
/usr/bin/fdfs_trackerd /etc/fdfs/storage.conf
#验证是否启动
ps -ef | grep storage
```

​ 3.验证是否可上传文件

```shell
#这个是查看FastDFS所有的可执行命令
ll /usr/bin/fdfs*
#验证之前需要修改client.conf配置文件
vim /etc/fdfs/client.conf
#常用配置
base_path=/home/fdfs/work/client #存放数据及日志文件 (路径文件夹必须存在)
tracker_server=本地IP:22122 #指定tracker服务 (必须修改)
http.tracker_server_port=9995 #http访问tracker程序端口
#利用 fdfs_test 命令验证，使用方式:/usr/bin/fdfs_test 客户端配置文件地址 upload 上传文件
/usr/bin/fdfs_test /etc/fdfs/client.conf  upload /etc/fdfs/client.conf
#不包error即成功 会返回很多信息  包括url 但是此时的url是没法访问下载的，我们需要集合nginx
```

​ 上传成功截图

​ ![上传成功截图](2.jpg)

配置完以上两个文件 并成功启动和测试 基本上一个单机版的 FastDFS 就完成 80%了接下来我们需要配置 nginx 完成下载。

#### 配置 nginx 模块

​ 1.安装依赖

```shell
 #依赖包下载安装
 yum -y install zlib zlib-devel openssl openssl--devel pcre pcre-devel
```

​ 2.解压 nginx 和 fast-nginx-module-master 程序包,下载 nginx 在这里不做阐述，去即[官网下载](http://nginx.org/en/download.html)即可。

```shell
 #解压nginx和fastdfs-nginx-module-master
 tar -zxvf /nginx-1.14.2.tar.gz
 unzip -o fastdfs-nginx-module-master.zip -d /usr/local
 #将/usr/local/fastdfs-nginx-module-master/src/下的mod_fastdfs.conf 复制到/etc/fdfs/下
 cp /usr/local/fastdfs-nginx-module-master/src/mod_fastdfs.conf /etc/fdfs
 #修改mod_fastdfs.conf
 store_path0=/home/fdfs/work/storage  #仓储路径 最好和storage配置一样
 storage_server_port=23000  #要和storage的port配置一样
 tracker_server=本地IP:22122 #指定tracker服务
 base_path=/home/fdfs/work/storage #存放日志文件和一些数据文件
 #为nginx添加模块
 ./configure --prefix=/usr/local/nginx/ --add-module=/usr/local/fastdfs-nginx-module-master/src/
 make
 make install #如果是第一次安装nginx  需要install  如果已经存在nginx  则不需要这一步
 #配置nginx.conf
 server {
 		listen 监听端口;
 		server_name 本机IP;

 		location /group1/M00/{
 			ngx_fastdfs_module;
 		}
 }

#启动Nginx (如果重启可以采用  平滑重启 -s reload的方式)
```

​ 3.验证下载

​ 上传文件后运用本机 ip+nginx 监听端口+group_name+remote_filename 的方式

grop_name 和 remote_filename 就是上面测试成功返回的信息中 画红线 的两个值

#### 结合 Java Spring Boot 框架 构建文件上传服务

这里我不做详细说明：提供一个别人的 github 项目给大家参考 [请点击这里参考](https://github.com/bojiangzhou/lyyzoo-fastdfs-java)
