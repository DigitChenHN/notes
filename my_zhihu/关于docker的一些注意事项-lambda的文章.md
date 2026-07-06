## 导言

***主要记录本人作为docker新手使用过程中的学习到的一些知识。作为笔记仅供个人参考，各部分之间大概不会有很强的联系。***

如果自己写过项目或者从github安装过某些[开源项目](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE&zhida_source=entity)，就知道要让在[开发环境](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83&zhida_source=entity)中能够成功运行的程序在其他机器上运行是非常麻烦的一件事。

而[容器化](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E5%AE%B9%E5%99%A8%E5%8C%96&zhida_source=entity)几乎完美地解决了这个问题，docker就是实现容器化的一种方式。容器化简单理解就是将运行该项目所需要的环境整个打包。打包好的这个文件就叫做镜像（image），其他人将镜像下载下来，使用docker就可以根据镜像在本地运行一个对应的容器（container），从而实现成功运行其中的项目程序的效果。

如果与[虚拟机](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E8%99%9A%E6%8B%9F%E6%9C%BA&zhida_source=entity)进行对比，就可以更好的理解容器化。虚拟机是将计算机的物理资源进行[抽象化](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E6%8A%BD%E8%B1%A1%E5%8C%96&zhida_source=entity)，划分为多个逻辑资源，每一个虚拟机都有自己的虚拟cpu、虚拟内存、[虚拟磁盘](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E8%99%9A%E6%8B%9F%E7%A3%81%E7%9B%98&zhida_source=entity)等；而容器化的概念中，各个容器是**共享一个操作系统内核**的，也就是说，比操作系统更加底层的资源也是直接共享的，比如cpu、内存等。这里说共享[操作系统内核](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=2&q=%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E5%86%85%E6%A0%B8&zhida_source=entity)，一般是[Linux系统内核](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=Linux%E7%B3%BB%E7%BB%9F%E5%86%85%E6%A0%B8&zhida_source=entity)，而根据**不同的[镜像文件](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E9%95%9C%E5%83%8F%E6%96%87%E4%BB%B6&zhida_source=entity)**生成的容器是可以基于不同的Linux发行版构建的。比如一个基于Debian，一个基于Alpine，那么容器中的系统库、工具、版本都是不一样的，这样一来就实现了在同一台机器上的运行不同的操作系统环境。

## 为简单项目构建docker镜像

假设使用python 3.11 写了一个简单的服务脚本（不涉及连接数据库之类的），为了使得另一台装有docker的机器能够方便地运行这个服务，就可以在**当前项目**目录下写一个**[Dockerfile](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=Dockerfile&zhida_source=entity)**，然后让这台机器上的docker按照Dockerfile这个脚本文件一步一步地配置环境。

具体来讲，一个简单的Dockerfile如下：

```text
# 使用轻量级的 Python 镜像作为基础
FROM python:3.11-slim

# 设置工作目录
WORKDIR /app

# 复制项目文件到容器中
COPY . .

# 安装依赖（假设你有 requirements.txt）
RUN pip install --no-cache-dir -r requirements.txt

# 设置容器启动时执行的命令
CMD ["python", "main.py"]
```

可以看到开头第一行`FROM python:3.11-slim`，个人项目一般都需要拉取一个基础的镜像作为构建自定义镜像的第一步，常见的基础镜像有系统镜像alpine、ubuntu，[编程语言](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E7%BC%96%E7%A8%8B%E8%AF%AD%E8%A8%80&zhida_source=entity)镜像python、node、[golang](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=golang&zhida_source=entity)，数据库镜像mongo、redis等等。

有了Dockerfile，就可以使用`docker build --tag name:tag`命令构建基于当前项目的名为name:tag的镜像。

有了项目镜像，就可以使用`docker run name:tag` 运行一个容器，这个容器中就包含了项目所需要的运行环境（对应版本的python、linux系统等等）以及[项目代码](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E9%A1%B9%E7%9B%AE%E4%BB%A3%E7%A0%81&zhida_source=entity)。

> 所谓基础镜像也是相对而言，比如编程语言镜像python中肯定也是包含了linux系统在里面的（当然大概率不是通过编写Dockerfile并在开头使用`FROM ubuntu`这样的方式 ）。如果你的项目实现的是一个比较基础的功能，那么也可以作为基础镜像被其他项目引入。

-   `docker pull name:tag`从镜像服务器拉取镜像
-   `docker rmi name:tag` 删除镜像
-   `docker ps`查看运行中的容器
-   `docker stop <container_id_or_name>`，停止容器
-   `docker rm <container_id_or_name>`，删除容器

## [数据卷](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%8D%B7&zhida_source=entity)

假设这个简单的python项目需要读写文件，在写代码的时候将数据保存在项目根目录下的data文件夹中，那么构建好容器并运行项目时，项目产生的数据文件会被存放在容器中的`/data/`下，如果删除容器，数据同样会被删除。因此一种更为灵活的方式是使用docker创建数据卷并挂载到容器中。

`docker volume create mydata`创建名为mydata的数据卷；

`docker run -d --name container_name -p 8080:80 -v mydata:/data image_name` 运行容器时挂载数据卷， 其中`-v`后面的参数格式为`数据卷名称:容器中的路径`，也可以使用`主机路径:容器中的路径`将主机目录挂载到容器中。这样一来，[删除容器](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=3&q=%E5%88%A0%E9%99%A4%E5%AE%B9%E5%99%A8&zhida_source=entity)就不会删除容器运行过程中产生的数据文件，从而实现数据永久化。

这里多提一个[端口映射](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E7%AB%AF%E5%8F%A3%E6%98%A0%E5%B0%84&zhida_source=entity)，即将容器内的端口映射到主机端口上，`-p`参数格式为`主机端口:容器端口`，并且可以使用多个`-p`实现多个端口映射。

## docker compose

那么如果你的项目是多个服务配合运行的，那么就需要运行多个镜像，比如需要一个运行项目后端服务的容器和一个运行redis数据库的容器配合使用，为了实现这个过程，容器与容器之间需要进行一些组合和编排，比如设置启动顺序、网络连接、设置依赖关系、配置[环境变量](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F&zhida_source=entity)、设置端口等等。为了让以上过程[自动化](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E8%87%AA%E5%8A%A8%E5%8C%96&zhida_source=entity)且完全可重复，就有了[docker compose](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=2&q=docker+compose&zhida_source=entity)。

docker compose主要是针对需要多镜像配合的项目，根据一个.yaml文件执行[容器组合](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E5%AE%B9%E5%99%A8%E7%BB%84%E5%90%88&zhida_source=entity)和编排。举例来说：

```text
version: '3'
services:
  web:
    build: ./web
    ports:
      - "8000:8000"
  redis:
    image: redis:alpine
```

这个项目使用到了自己编写的web服务和redis数据库，需要运行两个容器。

-   `[docker-compose build](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=docker-compose+build&zhida_source=entity)`：根据yaml文件构建镜像。以上面这个例子来讲，会按照`./web`路径下的Dockerfile构建镜像web；由于redis:alpine是官方镜像，不需要构建（如果本地没有会尝试拉取）
-   `[docker-compose up -d](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=docker-compose+up+-d&zhida_source=entity)`：启动服务并**后台运行**。启动所有服务的容器，启动容器是需要镜像的，所以如果镜像没有构建或者拉取，则会尝试自动构建或者拉取。
-   `[docker-compose down](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=docker-compose+down&zhida_source=entity)`：该命令用于停止并删除由`*docker-compose up*`启动的容器、网络和其他相关资源。
-   `docker-compose start / stop` :命令用于启动/停止由*docker-compose.yml*文件定义的所有[服务容器](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E6%9C%8D%E5%8A%A1%E5%AE%B9%E5%99%A8&zhida_source=entity)。它不会删除容器或相关的网络和卷，仅将容器状态更改为停止。**需要特别注意的是，如果项目使用了数据卷来存放相关的数据，如果使用`docker-compose down`，会将这些数据也一并删除。**

## docker 运维

docker的容器运行起来之后，可以理解为利用共享的操作系统内核运行了一个独立的操作系统，然后在操作系统上运行指定的各种依赖和主要的程序，因此理论上应该是可以像使用一个独立的操作系统一样使用各种命令进行管理的。

最主要的一条命令：`docker exec -it <container_name> 命令`，意味这在名为<container\_name>中执行交互式命令。

比如一个postgres的[数据库容器](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E5%BA%93%E5%AE%B9%E5%99%A8&zhida_source=entity)，想要连接到其中的数据库并实现增删改查，可以执行`docker exec -it thePostgres psql -U postgres dify`.其中`psql -U postgres dify`是postgres的客户端命令，该命令意味着指定用户名为`postgres`(默认的超级用户）连接到名为dify的数据库。

## Windows上的docker

Docker官方为Windows提供了[docker desktop](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=docker+desktop&zhida_source=entity)这个软件。

由于docker是需要Linux内核的，所以在Windows上运行docker desktop 实际上需要基于Windows上自带的wsl（Windows subsystem for linux）这个功能的。目前大多数Windows上的wsl是[wsl2](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=wsl2&zhida_source=entity)，wsl2本质上就是利用了Windows自带的[hyper-v](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=hyper-v&zhida_source=entity)创建了虚拟机，使原本的Windows也成为运行在hyper-v系统上的虚拟机。

以下总结的经验主要来源于在wsl2中的Linux发行版上直接安装和运行[docker容器](https://zhida.zhihu.com/search?content_id=259041541&content_type=Article&match_order=1&q=docker%E5%AE%B9%E5%99%A8&zhida_source=entity)的情况。（不太确定使用docker desktop和直接在wsl2中的某一个发行版上运行docker有什么区别，因此做个提醒）

## 容器内服务的通信

如果使用docker部署了一个容器运行了一个客户端A；然后在Windows上运行了一个服务B，试图接收客户端A的http请求，那么首先要求服务B运行的host为`0.0.0.0`而非`127.0.0.1`；其次在服务A中发送请求时，地址为`http://host.docker.internal`而非`http://localhost`或者`127.0.0.1`。

\=====待更新==========