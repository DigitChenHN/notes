# 使用云服务器搭建Web服务器

 **Author:** [lambda]

 **Link:** [https://zhuanlan.zhihu.com/p/27263730897]

## 引言  
这篇文章主要是从“有一个云服务器，如何让其可以响应客户端的请求，与客户端完成通讯？”的场景出发来学习网站服务器搭建的过程，其中会包括较多的学习理解作为个人记录，并非简单的流程式教程。

## 什么是Web服务器？  
通常我们讲到服务器都会联想为一台性能较高的、放在机房中时刻运行的计算机，但其实只有搭配上一系列的软件之后，在互联网上能够实现一定功能才是我们常说的服务器，比如游戏服务器、网站服务器。 

以下是搜索引擎对于网站服务器的简要介绍。其中值得不熟悉服务器的人群注意的是，服务器一般指的是机器上运行的、用来处理请求的软件，而非机器本身，像一些经常看到的相关名词如Nginx、Apache，就是所谓的服务器软件：


> **Web 服务器**，也称为**网站服务器**，是驻留在互联网上的一种计算机程序，主要功能是存储、处理和传递网页给客户端。Web 服务器可以处理浏览器等 Web 客户端的请求并返回相应的响应[[ref]](https://blog.csdn.net/HelloZEX/article/details/122810557)[[ref]](https://baike.baidu.com/item/WEB服务器/8390210)。  
> **Web 服务器的工作原理**  
> Web 服务器的工作原理可以分为以下几个步骤[[ref]](https://blog.csdn.net/HelloZEX/article/details/122810557)[[ref]](https://baike.baidu.com/item/WEB服务器/8390210)：  
> **连接过程**：通过 TCP 协议的三次握手建立与目标 Web 服务器的连接。  
> **请求过程**：用户代理（如浏览器）发起资源请求，通常是通过 URL（统一资源标志符）发起的。  
> **应答过程**：Web 服务器接收到请求后，按照 HTTP 协议处理请求并返回响应。  
> **关闭连接**：完成数据传输后，客户端和服务器关闭连接。  
> **Web 服务器的类型**  
> Web 服务器可以分为**静态 Web 服务器**和**动态 Web 服务器**[[ref]](https://developer.mozilla.org/zh-CN/docs/Learn\_web\_development/Howto/Web\_mechanics/What\_is\_a\_web\_server)：  
> **静态 Web 服务器**：由一个计算机（硬件）和一个 HTTP 服务器（软件）组成，传送的文件保持原样。  
> **动态 Web 服务器**：由静态 Web 服务器加上额外的软件（如应用服务器和数据库）组成，传送的文件在发送前会进行更新。  
> **常见的 Web 服务器软件**  
> 目前主流的 Web 服务器软件包括 Apache、Nginx 和 IIS[[ref]](https://blog.csdn.net/HelloZEX/article/details/122810557)[[ref]](https://baike.baidu.com/item/WEB服务器/8390210)。这些服务器软件各有特点，适用于不同的应用场景。  
> **Apache：**Apache 是世界上使用最广泛的 Web 服务器，具有高性能和跨平台的特点[[ref]](https://baike.baidu.com/item/WEB服务器/8390210)。  
> **Nginx：**Nginx 以其高并发处理能力和低资源消耗而著称，适用于高流量的网站[[ref]](https://baike.baidu.com/item/WEB服务器/8390210)。  
> **IIS：**IIS 是 Microsoft 提供的 Web 服务器，集成了多种服务组件，适用于 Windows 平台[[ref]](https://baike.baidu.com/item/WEB服务器/8390210)。  
> **总结**  
> Web 服务器是互联网的重要组成部分，负责处理客户端的请求并返回相应的网页内容。了解 Web 服务器的工作原理和常见的服务器软件，有助于更好地构建和维护网站[[ref]](https://blog.csdn.net/HelloZEX/article/details/122810557)[[ref]](https://baike.baidu.com/item/WEB服务器/8390210)[[ref]](https://developer.mozilla.org/zh-CN/docs/Learn\_web\_development/Howto/Web\_mechanics/What\_is\_a\_web\_server)。

一个拥有公网ip的计算机搭载了一系列的软件，就成了我们经常说的“服务器”。比如在相关话题中总能看到“LAMP”这个词汇，它指的是Linux、Apache、MySQL、PHP这一系列的“软件”，这些软件分别扮演了操作系统、web服务器、数据库服务器、网页开发语言的身份。 

## 粗糙地了解一个客户端与服务器通信的过程（非严谨定义，仅作理解）  
首先，不管是否具有操作系统，一台可以作为服务端的计算机往往首先需要有一个公网ip，公网ip是互联网上唯一标识一个网络接口的地址，只有通过公网ip才能被任何接入互联网的设备直接访问。

有了这个地址，客户端就可以向服务端发送请求（本质上也只是一大串0和1的数字信号，可以理解为**数据包**），但实际上一大串的0和1需要在规定好的语法下才具有特定的含义，这大概就是所谓的协议。上面说了ip地址唯一指向了某一个设备，而这个设备上又可以运行不同的程序（这其中有一部分程序被叫做服务），所以需要更深一级的的地址（门牌号）来标识，也就是所谓的端口，在端口上运行某个服务，或者说让某个服务去监听某个端口，当有数据包从客户端发送到这个端口时，特定的服务进程就接收该数据包，并且按照某种特定的协议去理解该数据包的内容，继而做出响应。 

有了以上的认知，对于开头的问题：“有一个云服务器，如何让其可以响应客户端的请求，与客户端完成通讯？”就可以有一个比较简单的方案。在作为服务端的计算机上运行一个服务（本质上就是一个程序），这个后端服务可以是一个简单的http服务器，令它监听特定的端口，当客户端向该端口发送网络请求时，服务就可以根据协议处理接收到的数据包，理解请求的内容，并且做出响应。

以下是一个使用python和Flask库编写的后端服务，使其监听计算机所有网络接口的5000端口，就可以实现当有使用get方法的http请求发送到5000端口时，服务做出响应，返回一个列表`my_list`：


```
from flask import Flask, jsonify

app = Flask(__name__) 
# 创建一个Flask实例，当这个脚本作为主程序运行时，__name__变量会被赋值为"__main__"，
# 使用该变量来构建实例是为了能够确定应用的根目录

@app.route('/list', methods=['GET'])
# 路由装饰器，将url路径‘/list’和下面这个函数get_list_fun绑定起来
def get_list_fun():
    # 这里可以根据请求的参数或其他逻辑来生成列表
    # 例如，从数据库中获取数据，或者基于请求头、查询参数等
    my_list = ["item1", "item2", "item3"]  # 示例列表
    return jsonify(my_list)  # 将列表作为JSON响应返回

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)  # 在所有网络接口上监听5000端口
```
以上代码在云服务器上运行即可启动一个监听端口5000的服务（注意：还要根据服务器上的Linux版本在防火墙开放对应端口，阿里云的话还需要设置安全组），当有设备对http://云服.务器.的公.网ip:5000/list发送get请求的时候，云服务器就会根据`get_list_fun`函数响应。 

此外，还可以考虑将其设置为`systemd` 从而实现长期后台运行：

* 首先创建一个服务单元文件，一般在`/etc/systemd/system`目录下，文件命名为my\_script.service


```
 [Unit]
Description=My Python Script
After=network.target

[Service]
Type=simple
User=your_username
ExecStart=/usr/bin/python3 /path/to/your_script.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
* 重新加载 `systemd` 配置：


```
sudo systemctl daemon-reload
```
* 启动服务并设置服务的开机启动


```
sudo systemctl start my_script.service
sudo systemctl enable my_script.service
```
一个成熟的web服务器除了像以上实现正常请求处理外，还要考虑性能、安全等因素，比如以上的操作会将5000端口对外开放，这可能会是一个安全的隐患；而这样问题往往能够通过使用成熟的服务套件，比如LAMP或者LNMP解决。 比如HTTP和HTTPS的默认端口分别是80和443，我们平常使用客户端发送http请求都不需要指定端口，当请求发送到服务端的80端口时，由Nginx这种所谓的“**反向代理服务器**”监听80端口并接收请求，进而由它来实现和后端服务的沟通。（或者说通过反向代理服务器进行转发到5000端口，这样就不需要将5000端口暴露在防火墙外了）

此外，这种简单的实现中，由于请求是http协议的，在互联网上传输的数据内容是未经过加密的，被拦截之后将明文展示给拦截者从而导致信息的泄露。现在很多网站使用的都是https，而配置https则涉及到了更多的内容，比如SSL证书；而反向代理服务器可以提供SSL/TLS加密、负载均衡、缓存和访问控制等附加功能。

