# docker搭建lnmp环境

<!-- TOC -->

- [docker搭建lnmp环境](#docker%E6%90%AD%E5%BB%BAlnmp%E7%8E%AF%E5%A2%83)
  - [一、Dockerfile定制镜像](#%E4%B8%80dockerfile%E5%AE%9A%E5%88%B6%E9%95%9C%E5%83%8F)
  - [二、docker-compose](#%E4%BA%8Cdocker-compose)
  - [三、docker-compose编排lnmp环境](#%E4%B8%89docker-compose%E7%BC%96%E6%8E%92lnmp%E7%8E%AF%E5%A2%83)
    - [1、mysql](#1mysql)
    - [2、redis](#2redis)
    - [3、mongo](#3mongo)
    - [4、nginx](#4nginx)
    - [5、php](#5php)
    - [6、完整版](#6%E5%AE%8C%E6%95%B4%E7%89%88)
  - [四、参考](#%E5%9B%9B%E5%8F%82%E8%80%83)

<!-- /TOC -->

有收获的话请**加颗小星星**，没有收获的话可以 **反对** **没有帮助** **举报**三连

- [代码仓库](https://github.com/OMGZui/lnmp)
- [docker搭建lnmp环境](http://moll.omgzui.top/article/9)

## 一、Dockerfile定制镜像

```bash
# FROM 指定基础镜像
FROM 镜像

FROM php:7.2-fpm

# RUN 执行
RUN <命令>
or
RUN ["可执行文件", "参数1", "参数2"]

RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
RUN ["php", "-S", "0.0.0.0:8080"]

# COPY 复制文件
COPY <源路径>... <目标路径>

COPY swoole-4.2.10.tgz /home
COPY nginx.conf /etc/nginx/nginx.conf

# ADD 复制文件或目录，如果是.tgz，会被解压缩
ADD <源路径>... <目标路径>

ADD nginx.conf /etc/nginx/nginx.conf

# CMD 容器启动
CMD echo $HOME => CMD [ "/bin/sh", "-c", "echo $HOME" ]

CMD [ "redis-server", "/usr/local/etc/redis/redis.conf" ]

# ENTRYPOINT 入口点
ENTRYPOINT ["docker-entrypoint.sh"]
存在 ENTRYPOINT 后，CMD 的内容将会作为参数传给 ENTRYPOINT

# ENV 环境变量
ENV <key> <value>

ENV MYSQL_ROOT_PASSWORD root

# ARG与ENV差不多
ARG 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的

ENV MYSQL_ROOT_PASSWORD root

# VOLUME 匿名卷
VOLUME ["<路径1>", "<路径2>"...]

VOLUME ["/data"]

# EXPOSE 暴露端口
EXPOSE <端口1> [<端口2>...]

EXPOSE 80 443

# WOEKDIR 指定工作目录，进入容器后的落地目录
WORKDIR <工作目录路径>

WORKDIR /var/www

# USER 指定当前用户
USER <用户名>

USER root
```

## 二、docker-compose

详细请查看 https://docker_practice.gitee.io/compose/

- 服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

- 项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

## 三、docker-compose编排lnmp环境

### 1、mysql

这里我们使用了mysql5.5版本，没其它用意，相比5.7以上版本，占内存和硬盘最小的一个版本

我们准备了一个`my.cnf`作为额外配置，这里我修改了数据库的时区

```cnf
[mysqld]

default-time-zone = '+8:00'

```

```Dockerfile
FROM mysql:5.5

COPY my.cnf /etc/mysql/conf.d

EXPOSE 3306
```

### 2、redis

我们使用准备的配置文件`redis.conf`覆盖容器默认启动的配置文件，修改了`ip绑定`和`密码`

```conf
bind 0.0.0.0
requirepass root
```

```Dockerfile
FROM redis:latest

COPY redis.conf /usr/local/etc/redis/redis.conf

CMD [ "redis-server", "/usr/local/etc/redis/redis.conf" ]

EXPOSE 6379
```

### 3、mongo

mongodb我们没有特殊处理

```Dockerfile
FROM mongo:latest

EXPOSE 27017
```

### 4、nginx

我们准备了一份`nginx.conf`和虚拟目录`conf.d`，为了以后可以动态的配置网站的代理和负载均衡

还有一个日志目录，放在外层`logs`目录里面，记录nginx的访问日志

特别注意的是`fastcgi_pass php:9000;`而不是`fastcgi_pass 127.0.0.1:9000;`，目前自己也没明白

```Dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
```

### 5、php

php算是这里面最难搞定的，因为我们需要额外的添加php扩展，虽然php的docker官方提供了`docker-php-ext-configure`, `docker-php-ext-install`, `docker-php-ext-enable`，还是有些扩展需要通过`手动编译`或者`pecl`安装

由于pecl官网下载慢，我们事先下载好了几个需要的扩展

php-fpm用的是debian的linux系统，下载也很慢，我们备用了阿里云的镜像`sources.list`

我们还准备了php的默认配置`php.ini`和`opcache.ini`

比如swoole扩展安装，记得安装包用完后清理，还有得用`COPY`命令，`ADD`会解压缩

```Dockerfile
# swoole
COPY swoole-4.2.10.tgz /home
RUN pecl install /home/swoole-4.2.10.tgz && \
    docker-php-ext-enable swoole && \
    rm /home/swoole-4.2.10.tgz
```

### 6、完整版

```yml
version: '3'

networks: 
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes: 
  mysql: 
    driver: local
  mongo:
    driver: local
  redis:
    driver: local

services: 
  php:
    build: ./php
    volumes: 
      - ${WORKER_DIR}:/var/www
    ports: 
      - 9100:9000
    depends_on: 
      - mysql
      - redis
      - mongo
    networks: 
      - backend

  nginx:
    build: ./nginx
    volumes: 
      - ${WORKER_DIR}:/var/www
      - ./logs/nginx:/var/log/nginx
      - ./nginx/conf.d:/etc/nginx/conf.d
    ports: 
      - 8000:80
    depends_on: 
      - php
    networks: 
      - frontend
      - backend

  mysql:
    build: ./mysql
    environment: 
      - MYSQL_ROOT_PASSWORD=root
    volumes: 
      - ${DATA_PATH}/mysql:/var/lib/mysql
    ports: 
      - 3310:3306
    networks: 
      - backend
  
  mongo:
    build: ./mongo
    environment: 
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=root
    ports: 
      - 27010:27017
    volumes: 
      - ${DATA_PATH}/mongo:/data/db
    networks: 
      - backend
  
  redis:
    build: ./redis
    volumes: 
      - ${DATA_PATH}/redis:/data
    ports: 
      - 6310:6379
    networks: 
      - backend

```

## 四、参考

- [Docker — 从入门到实践](https://docker_practice.gitee.io/)
- [laradock](https://github.com/laradock/laradock)
- [Docker在PHP项目开发环境中的应用](https://avnpc.com/pages/build-php-develop-env-by-docker)