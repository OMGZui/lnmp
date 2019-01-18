# 简明docker教程

<!-- TOC -->

- [简明docker教程](#简明docker教程)
    - [一、什么是docker](#一什么是docker)
    - [二、docker与虚拟机比较](#二docker与虚拟机比较)
    - [三、安装docker](#三安装docker)
    - [四、基本概念](#四基本概念)
        - [1、镜像](#1镜像)
        - [2、容器](#2容器)
        - [3、数据卷](#3数据卷)
        - [4、挂载](#4挂载)
    - [五、参考资料](#五参考资料)

<!-- /TOC -->

有收获的话请**加颗小星星**，没有收获的话可以 **反对** **没有帮助** **举报**三连

- [代码仓库](https://github.com/OMGZui/lnmp)
- [简明docker教程](http://moll.omgzui.top/article/8)

## 一、什么是docker

Docker是一个开放源代码软件项目，让应用程序布署在软件货柜下的工作可以自动化进行，借此在Linux操作系统上，提供一个额外的软件抽象层，以及操作系统层虚拟化的自动管理机制。

Docker利用Linux核心中的资源分离机制，例如cgroups，以及Linux核心名字空间（namespaces），来创建独立的容器（containers）。这可以在单一Linux实体下运作，避免启动一个虚拟机造成的额外负担。Linux核心对名字空间的支持完全隔离了工作环境中应用程序的视野，包括进程树、网络、用户ID与挂载文件系统，而核心的cgroup提供资源隔离，包括CPU、存储器、block I/O与网络。从0.9版本起，Dockers在使用抽象虚拟是经由libvirt的LXC与systemd - nspawn提供界面的基础上，开始包括libcontainer库做为以自己的方式开始直接使用由Linux核心提供的虚拟化的设施，

上面都是废话，简言之Docker的思想来自于集装箱，集装箱解决了什么问题？在一艘大船上，可以把货物规整的摆放起来。并且各种各样的货物被集装箱标准化了，集装箱和集装箱之间不会互相影响。那么我就不需要专门运送水果的船和专门运送化学品的船了。只要这些货物在集装箱里封装的好好的，那我就可以用一艘大船把他们都运走。

## 二、docker与虚拟机比较

|特性|容器|虚拟机|
|-|-|-|
|启动|秒级|分钟级|
|硬盘使用|一般为 MB|一般为 GB|
|性能|接近原生|弱于|
|系统支持量|单机支持上千个容器|一般几十个|

## 三、安装docker

我自己用的是Docker for Mac

其它系统可以参考 http://docker_practice.gitee.io/install/

## 四、基本概念

- 镜像（Image）

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

- 容器（Container）

镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

- 仓库（Repository）

镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry 就是这样的服务。

一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。

### 1、镜像

```bash
# 获取镜像
docker pull ubuntu:14.04

# 以ubuntu:14.04镜像为基础启动并运行一个容器
docker run -it --rm \
    ubuntu:14.04 \
    bash
-it：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
--rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。
ubuntu:14.04：这是指用 ubuntu:14.04 镜像为基础来启动容器。
bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash。

# 列出镜像
docker image ls
docker images

# 镜像占用
docker system df

# 清楚悬挂镜像
docker image prune

# 删除镜像
docker image rm
docker rmi

```

### 2、容器

```bash
# 启动以守护模式创建的名字为demo-u的容器，并以交互模式进入容器
docker run --name demo-u -t -i -d ubuntu:14.04 bash

# 运行后就可以通过ID或名字进入容器，并输出hello world
docker exec -it demo-u /bin/sh -c "echo hello world"

# 查看运行中的容器
docker container ls
docker ps

# 所有容器
docker container ls -a
docker ps -a

# 查看容器日志
docker container logs demo-u
docker logs demo-u

# 终止容器
docker container stop demo-u
docker stop demo-u

# 启动容器
docker container start demo-u
docker start demo-u

# 重启容器
docker container restart demo-u
docker restart demo-u

# 进入容器，退出后容器也停止
docker attach demo-u

# 导出容器
docker export

# 导入容器
docker import

# 删除容器
docker container rm
docker rm

# 清除所有容器
docker container prune

```

### 3、数据卷

```bash
# 创建数据卷
docker volume create

# 列出数据卷
docker volume ls

# 删除数据卷
docker volume rm

# 清除没用的数据卷
docker volume prune

```

### 4、挂载

也就是目录共享，两种方式：

- -v
- --mount 推荐

```bash
# 使用php本地服务器查看php环境，加载主机的 ~/web 目录到容器的 /var/www/web目录
mkdir -p ~/web && cd ~/web && echo "<?php phpinfo();" > index.php
docker run -d \
    --name web \
    -p 8080:8080 \
    --mount type=bind,source=`pwd`,target=/var/www/web \
    php:7.2-fpm \
    /bin/sh -c "cd /var/www/web && php -S 0.0.0.0:8080"

或者

docker run -d \
    --name web \
    -p 8080:8080 \
    -v `pwd`:/var/www/web \
    php:7.2-fpm \
    /bin/sh -c "cd /var/www/web && php -S 0.0.0.0:8080"

打开浏览器 0.0.0.0:8080

# 查看数据卷
docker volume inspect web
```

## 五、参考资料

- [Docker — 从入门到实践](https://docker_practice.gitee.io/)