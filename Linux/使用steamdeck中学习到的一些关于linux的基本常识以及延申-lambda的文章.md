---
id: "25443657804"
title: "使用steamdeck中学习到的一些关于linux的基本常识以及延申"
author: "lambda"
type: zhihu-article
source: "https://zhuanlan.zhihu.com/p/25443657804/preview?comment=0&catalog=0"
created: "2025-02-21 20:39"
updated: "2026-05-14 20:41"
downloaded: "2026-05-14"
---
1.  常见的软件后缀有.rpm、.deb，其中rpm是redhat package manager的缩写，对应red hat linux系列的系统，比如centos；而.deb是debian，对应的是Debian系列的系统，比如Debian、Ubuntu。这些文件本质上是压缩文件，一般包含了软件的二进制文件、依赖信息、配置文件等。对应的archlinux上的软件后缀.pkg.tar.zst。
2.  常见的包管理工具：arch linux——pacman、flatpak；debian——apt（advanced packaging tool）、dpkg（package manager for Debian）；redhat linux——yum（yellow update manager）、RPM（redhat package manager）。
3.  其中apt和dpkg虽然都是Debian系列的包管理工具，但是二者也并不是平行的，一般dpkg用来安装已经在本地的.deb文件，而apt是能够直接从远程仓库下载软件，并且自动解决各种依赖问题。相应的，RPM和yum也不是平行关系，RPM是redhat公司开发的用来管理其linux系统中各种软件依赖关系的管理程序，而yum是它的前端工具，往往也是用来从远程仓库中下载安装软件并且自动解决复杂的依赖关系。
4.  说到远程仓库，arch linux的软件仓库叫做arch user repository （AUR），使用pacman安装软件就是通过该仓库。相应地，使用apt和yum都是在对应的软件仓库中去寻找软件，并且国内使用这些软件工具往往是为其配置一个对应软件仓库的国内镜像。
5.  debtap可以将deb转换成arch linux可以使用的软件格式。

## 在个人Linux机器上常用的Linux知识

关于Linux系统性地学习可以参看教程[Linux 教程 | 菜鸟教程](http://link.zhihu.com/?target=https%3A//www.runoob.com/linux/linux-tutorial.html)，由于教程内容比较多，一开始不一定都用得到，而且还有许多知识教程中不涉及，需要广泛地使用搜索引擎和ai学习，因此简单学习基础之后快速上手是比较重要的。以下记录的是本人开始接触Linux之后常用的一些命令和经验。

### 创建用户相关

一般都建议创建一个区别于root的用户。   

`adduser 用户名`可以跟着系统指引创建用户，包括设置密码和用户目录等等。  

`useradd 用户名`则需要手动配置密码和用户目录等。  

创建用户之后，一般就用这个用户登录系统，但是很多时候需要执行管理员权限级别的命令，所以需要将新建的用户加入到sudo组中，最简单的方法就是`usermod -a -G sudo 用户名`。  

root使用`userdel name`命令删除用户；使用`passwd name`设置用户的密码。   

如果想查看当前系统中有什么用户，可以考虑将`/etc/passwd`中的信息打印出来查看。    

### 用户权限相关

  - `chmod +x xxx`，给xxx文件添加执行权限。    
  - `sudo`这个命令非常常见，应该是【super user do】的缩写，这个命令是为了让普通用户能够临时以超级管理管的身份运行相关的命令。  

### 磁盘与文件相关情况

-   `du -sh` 查看当前目录大小，其中*\-h*：以KB、MB、GB为单位显示大小，提高可读性。-s：仅显示总计大小。
-   `du -sh ./*` 列出当前目录下每个子项的大小。  
-   `df -h`以方便阅读方式显示系统磁盘使用情况。

## 管理系统服务（系统初始化）

**systemd**是一种用于 Linux 系统的初始化系统和服务管理器，它是传统的 SysVinit 和 Upstart 的替代品。systemctl是它的一部分，常常用systemctl来管理服务，包括后台运行、开机自启等等的实现。

-   `systemctl status xxx` 查看xxx服务的运行情况
-   `systemctl start / stop xxx` 启动/关闭 xxx服务
-   `systemctl enable xxx` 设置xxx服务的开机自启

如果是自己写的脚本服务，想要通过systemctl进行管理，需要写一个systemd的单元文件（.service后缀的文本文件）。

### 服务文件存放位置

systemd 的服务文件通常存放在以下两个目录中：

-   **系统级服务**： `/etc/systemd/system/`  
    适用于全局服务，优先级最高。
-   **用户级服务**： `~/.config/systemd/user/`  
    适用于当前用户的服务（需启用用户级 systemd）。

推荐将服务文件放在 `/etc/systemd/system/` 目录下。

### 服务文件的基本结构

一个典型的 `.service` 文件包含以下部分：

```text
[Unit]
Description=描述服务的用途
After=network.target  # 定义服务的启动依赖（如网络就绪）

[Service]
Type=simple           # 服务类型（simple/forking/oneshot）
User=用户名           # 运行服务的用户
Group=用户组          # 运行服务的用户组
WorkingDirectory=/path/to/your/app  # 服务的工作目录
ExecStart=/usr/bin/python3 /path/to/your/app.py  # 启动命令
Restart=always        # 服务崩溃时自动重启
RestartSec=5          # 重启间隔（秒）
StandardOutput=journal  # 日志输出到 journald
StandardError=journal   # 错误输出到 journald
Environment=KEY=value  # 设置环境变量

[Install]
WantedBy=multi-user.target  # 定义服务启动级别
```

一般修改、新建一个单元服务之后需要重新`sudo systemctl daemon-reload`重新加载systemd单元文件。  

### 服务日志

在 Linux 系统中，使用`systemctl`启动的服务（如 Flask 应用）默认会将标准输出（stdout）和标准错误（stderr）重定向到系统的日志服务（通常是`journald`）。  

journalctl是systemd中用来管理日志的工具。  

```text
# 查看完整的服务日志
journalctl -u xxxx.service

# 实时跟踪日志（类似 tail -f）
journalctl -u xxxx.service -f

# 查看最近 100 行日志
journalctl -u xxxx.service -n 100

# 按时间过滤（例如查看今天的日志）
journalctl -u xxxx.service --since today
```

  

\-----代补充

此外还有一些其他的系统服务管理器，比如在wsl2中直接安装的ubuntu24.04中，服务管理器默认不是使用的systemd，wsl2应该已经支持systemd，具体设置方法 ----待补充-----。

其默认的服务管理器是---待补充----

可以使用`service`命令来代替systemd中的`systemctl`命令。

-   `sudo service xxx start` 启动xxx

## 网络相关

netstat是一个非常常用的工具，可以用来查看网络连接、路由表、接口统计信息等。

经常搭配grep（global regular expression）

## grep（global regular expression）
grep是一个利用正则表达式进行文本搜索的命令行工具。正则表达式最早就是通过Unix中的工具软件如grep、sed等推广开来的。  
grep的基本语法如下：
```
grep [选项] 模式 [文件]
```
其中，模式即用[正则表达式](/regex.md)进行匹配的模式。  
虽然正则表达式是跨语言的文本匹配模式，但是[不同语言和工具对正则表达式的支持不同](/regex.md/#高级特性)。对于grep，一般有三种模式：基本正则表达式（BRE）、扩展正则表达式（ERE）和Perl兼容正则表达式（PCRE）。默认情况下，grep使用基本正则表达式（BRE），选项`-E`可以启用扩展正则表达式（ERE），选项`-P`可以启用Perl兼容正则表达式（PCRE）。  



由于在steamos上使用pacman安装的软件，在系统更新之后都会被删除，所以需要手动重新安装。在此做一个记录：

1.  n2n
2.  mihomo是直接下载的便携版本并将其放到`/usr/local/bin`路径下，这个路径默认是环境变量中，所以放入后就可以直接执行。但是系统更新之后该目录会重置为只读，因为steamos整个系统就是只读的，所以需要使用命令`sudo steamos-readonly disable`解除该状态。
3.  在 /etc路径下创建一个enviroment文件，用于解决fcitx5安装并配置pinyin之后无法使用拼音输入法内容如下（这个似乎在更新系统后没有变化）：

```text
LANG=en_US.UTF-8
LC_CTYPE=zh_CN.UTF-8
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

