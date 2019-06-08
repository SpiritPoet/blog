---
title: 利用Vultr搭建Shadowsocks
date: 2019-06-08 09:17:16
categories:
    - 工欲善其事必先利其器
tags:
    - 利器-科学上网
---

# 利用 Vultr 搭建 Shadowsocks（科学上网）

![logo](1.png)

<!-- more -->

## 搭建步骤

> 1.配置服务器 2.在服务器上安装 Shadowsocks 3.安装 Shadowsocks 客户版 4.开启 TCP BBR 算法

### 配置服务器

1.创建[Vultr](https://www.vultr.com/)账号
![Alt Text](2.png)

2.账户充值(支付宝可以支付 根据自身条件选择)
![Alt Text](8.png)

3.创建服务器
![Alt Text](3.png)

- 选择服务器所在地
  ![Alt Text](4.png)
- 选择服务器操作系统
  ![Alt Text](5.png)
- 选择服务 (这里如果只是上上网,\$2.5 的足够)
  ![Alt Text](6.png)
- 选择完服务后其他的可以忽略 直接点击最下面右下角的 Deploy
- 创建完成后 Server 里显示 正在初始化 等 1 分钟即可
  ![Alt Text](7.png)
- 点击...进入管理界面，记住三个箭头的数据 后面有用！
  ![Alt Text](10.png)
  ![Alt Text](11.png)

### 安装 Shadowsocks

1.首先，下载 [Secure Shell Client](http://ultra.pr.erau.edu/~jaffem/tutorial/SSH_secure_shell_client.htm) 2.通过 Shell Client 连接服务器

> 1.输入 IP 和 UserName
> ![Alt Text](14.png) 2.输入 Password
> ![Alt Text](13.png)

3.运用脚本安装

> 1.运行如下代码

```
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

> 2.下面会出现让你选择脚本(笔者直接回车，默认 Python 版),然后让你设置连接密码和端口(笔者随便用了一个密码 123456，端口 2018)。接着回车等待安装完成，这里时间稍微有点长

```
Which Shadowsocks server you'd select:
1) Shadowsocks-Python
2) ShadowsocksR
3) Shadowsocks-Go
4) Shadowsocks-libev
Please enter a number (Default Shadowsocks-Python):

You choose = Shadowsocks-Python

Please enter password for Shadowsocks-Python
(Default password: teddysun.com):123456

password = 123456

Please enter a port for Shadowsocks-Python [1-65535]
(Default port: 11912):2018
```

> 3.安装完成提示如下

```
Welcome to visit:https://teddysun.com/486.html
Enjoy it!
[root@vultr etc]#
```

> 4.校验是否启动 Shadowsocks

```
[root@vultr /]# ./etc/init.d/shadowsocks-python status
Shadowsocks (pid 17336) is running...
```

> 5.关于脚本的命令以及如何配置多用户，更改 config 文件等请阅读[秋水逸冰 Shadowsocks 一键安装脚本（四合一）](https://teddysun.com/486.html)

### 安装 Shadowsocks 客户版

> 1.[常规版](https://github.com/shadowsocks/shadowsocks-windows/releases)【推荐】 2.[SSR 版](https://github.com/shadowsocksrr/shadowsocksr-csharp/releases)

#### 科学上网

> 1.如果记不得一开始安装时输入的密码和端口，请输入以下命令查看:

```
[root@vultr ~]# vi /etc/shadowsocks-python/config.json
{
    "server":"0.0.0.0",
    "server_port":2018,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"123456",
    "timeout":300,
    "method":"aes-256-gcm",
    "fast_open":true
}

```

> 2.配置客户端

- 1.输入 IP、端口和密码 这里要特别注意密码下面的加密哪一项 一定要和你 config.json 里的 method 的值对应 图中和我上面代码里的不对应所以是错误的 一定不要按图中的加密方式配置 要对应你的 config
  ![Alt Text](12.png)
- 2.右击程序小飞机图标 勾选上启动系统代理就可以上网了
  ![Alt Text](15.png)
- 3.系统代理模式有两种，PAC 和全局，PAC 是部分网页通过 ss，全局是所有流量都通过 ss（比如打游戏就要开全局）
- 4.至此属于你个人的 Shadowsocks 服务就搭建好了

### 开启 BBR 加速

> 1.什么是 BBR?

TCP BBR 致力于解决两个问题 1.在一定丢包率的网络链路上充分利用带宽 2.降低网络链路上的 Buffer 占用率，从而降低延迟

> 2.为什么开启 BBR?

就是让你更爽

> 3.如何开启 TCP BBR ?

我只提供出 CentOS7 的开启方法 其他系统 [百度一下 你就知道](https://www.baidu.com/)

- 1.首先更新系统，输入一下命令

```
 yum update -y
```

    直至出现 Complete!

- 2.安装内核

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

    ```
    rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
    ```
    ```
    yum --enablerepo=elrepo-kernel install kernel-ml
    ```

- 3.检查是否已安装

```
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
```

- 返回如下信息

```
0 : CentOS Linux (4.16.0-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux 7 Rescue 40f3bcfef5a2446abf07411db6ef7e14 (3.10.0-693.21.1.el7.x86_64)
2 : CentOS Linux (3.10.0-693.21.1.el7.x86_64) 7 (Core)
3 : CentOS Linux (3.10.0-693.11.6.el7.x86_64) 7 (Core)
4 : CentOS Linux (3.10.0-693.el7.x86_64) 7 (Core)
5 : CentOS Linux (0-rescue-c73a5ccf3b8145c3a675b64c4c3ab1d4) 7 (Core)
```

- 设置默认内核

```
 grub2-set-default 0
```

- 重启

```
reboot
```

- 编辑系统配置文件

```
vi /etc/sysctl.conf
```

- 添加或删除以下两行(注：按 i 进入编辑模式，编辑好后按 esc 退出编辑模式，再按 shift + zz 快速保存返回)

```
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

- 生效配置文件

```
sysctl -p
```

- 检查 bbr 算法是否开启

```
lsmod |grep bbr
```

如果返回信息中有 tcp_bbr 就是开启成功！

**至此全部搭建成功！**
