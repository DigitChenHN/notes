# 在云服务器上安装和使用Nginx

 **Author:** [lambda]

 **Link:** [https://zhuanlan.zhihu.com/p/27660225122]

## 引言  
Nginx作为非常成熟web服务器软件，可以实现如反向代理、SSL加密等各种功能。现在的主流云服务器应该都支持一键安装Linux+Nginx+MySQL+PHP（LNMP），为了更深入地了解该软件的功能并做一个记录，本文将主要展示在阿里云服务器上（Ubuntu 22.04.5）安装Nginx并实现一些基本的功能。

## 安装  
使用apt安装：


```
apt-get update 
apt-get install nginx
```
![]((20250313)在云服务器上安装和使用Nginx_lambda/v2-81480da771790e5f061db73489f60905_1440w.jpg)  
提示有些服务需要重启，看了下其中有我运行的一个简单的python写的服务。但是应该没有什么影响，所以回车继续。 

打印nginx版本测试是否安装成功：


```
nginx -v 
```
运行nginx：


```
systemctl start nginx 
```
接着在阿里云的安全组中开放80端口，这时候就可以通过云服务器的公网ip访问到Nginx了。 

![]((20250313)在云服务器上安装和使用Nginx_lambda/v2-cf618e47b905a03ee392be53e57e90d6_1440w.jpg)  
## 使用nginx设置反向代理到code-server  
### 安装并运行code-server  
阿里云服务器可以直接用官方提供的安装脚本：


```
curl -fsSL https://code-server.dev/install.sh | sh
```
安装的是4.96.4，因为最新版本的4.97.2中似乎存在某些bug，导致本地浏览器在运行code-server构建的网页js文件时报错。

安装完成之后需要修改配置文件`~/.config/code-server/config.yaml`：


```
bind-addr: 0.0.0.0:xxxx
auth: password
password: xxxxxxxx
cert: false
```
其中bind-addr将xxxx改为让code-server监听的端口号， password为登陆时要设置的密码。 

这样完成后就可以直接在控制台输入code-server来运行了。

也可以考虑使用systemd来后台运行，需要构建一个`/etc/systemd/system/code-server.service`文件，其中的内容为：


```
[Unit]
Description=code-server
After=network.target

[Service]
Type=simple
User=root # 用户名称，如果是管理员那就是root
ExecStart=/usr/bin/code-server --config /root/.config/code-server/config.yaml 
# 如果是系统管理员并且是用官方脚本安装的code-server，那就是以上这个路径，如果不是也有可能需要修改
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
### 配置nginx为code-server这个服务器软件的反向代理服务器  
首先根据Nginx官方的文档先了解一些nginx常用的命令：

1. nginx ——运行nginx
2. nginx -s stop ——停止nginx
3. nginx -s quit ——优雅地停止nginx
4. nginx -s reload ——重启nginx，当nginx在运行中并且更改了配置文件，记得用这个命令重新加载配置

配置nginx主要就是修改其配置文件，根据Nginx的官方文档[https://nginx.org/en/docs/beginners\_guide.html#proxy](https://nginx.org/en/docs/beginners\_guide.html#proxy)，可以知道其配置文件的结构主要由指令（directive）构成，指令分为简单指令（simple directives）和块指令（block directives），简单指令由名称和参数以空格分开组成，并以“；”结尾；而块指令结构相同但是是以花括号结尾，如果块指令中有其他指令，则称为context，比如下面展示的http就是一个所谓的context，常见的context还有event、server和location等。Nginx是由各种模块组成的，而模块由这些指令进行控制。比如server指令是用来创建和控制一个虚拟服务器的。对于nginx的介绍可以参看视频：

[Nginx入门必须懂3大功能配置 - Web服务器/反向代理/负载均衡\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1TZ421b7SD/?spm\_id\_from=333.337.search-card.all.click&vd\_source=d82de55cfde970cdf86016bef2c6de4e)  
nginx的一个常见用途是将其设置为代理服务器，在我们的例子中，这意味着服务器（nginx）接收请求，将它们传递给被代理服务器（code-server），从中检索响应，并将其发送给客户端。

nginx可以实现http请求的转发，如果将其设置为code-server的代理服务器，就不需要对外开放code-server的端口，只需要发送常规的http请求给云服务器的80端口（http的默认端口），就可以通过nginx的代理访问到code-server。

修改nginx配置`/etc/nginx/nginx.conf`：（假设code-server的绑定的端口为1234）


```
http {
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on; 

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # include /etc/nginx/conf.d/*.conf;
    # include /etc/nginx/sizes_enabled/*;
    # 这个默认配置一定要注释掉，因为它会从指定文件中引入另一个server块然后覆盖掉我们下面写的server块
    # 以上内容是默认配置中存在的，因为目前还不太了解具体作用，所以保留

    # 要设置一个反向代理服务器，需要添加以下内容
    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://localhost:1234; 
        }

        # error_page 500 502 503 504 /50x.html;
        # location = /50x.html {
        #     root /usr/share/nginx/html;
        # }
    }
}
```
在默认的布局文件的基础上（如果默认布局文件与上面的不同，也可以新建一个新的空白文件，然后将上述内容写入），首先添加一个server块指令，然后将用#将`include /etc/nginx/sizes_enabled/*;` 注释掉，由于该行语句会从指定文件中引入一个server块，从而覆盖我们下面写的块，所以需要注释掉才能使我们添加的server指令生效。

这样设置之后，通过公网ip就可以访问到code-server了。但是会出现`Error: WebSocket close with status code 1006`的报错。原因是nginx在代理时没有对websocket进行代理。解决办法：利用Upgrade协议头机制将连接从HTTP连接升级到WebSocket连接。具体配置就是将上面的location块指令补充为：


```
location / {
            proxy_pass http://localhost:1234; 
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection upgrade;
            proxy_set_header Accept-Encoding gzip;
}
```
