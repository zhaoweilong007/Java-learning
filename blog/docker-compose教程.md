<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [docker-compose使用](#docker-compose%E4%BD%BF%E7%94%A8)
  - [简介](#%E7%AE%80%E4%BB%8B)
  - [安装及使用](#%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BD%BF%E7%94%A8)
  - [命令说明](#%E5%91%BD%E4%BB%A4%E8%AF%B4%E6%98%8E)
  - [compose模块文件](#compose%E6%A8%A1%E5%9D%97%E6%96%87%E4%BB%B6)
  - [docker-omcpose常用命令](#docker-omcpose%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4)
    - [模板文件简介](#%E6%A8%A1%E6%9D%BF%E6%96%87%E4%BB%B6%E7%AE%80%E4%BB%8B)
      - [image](#image)
      - [build](#build)
        - [context](#context)
        - [dockerfile](#dockerfile)
      - [command](#command)
      - [container_name](#container_name)
      - [depends_on](#depends_on)
      - [ports](#ports)
      - [extra_hosts](#extra_hosts)
      - [volumes](#volumes)
      - [expose](#expose)
      - [links](#links)
      - [net](#net)
      - [dns](#dns)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# docker-compose使用

## 简介

compose项目是官方的开源项目，负责实现对docker镜像集群的快速编排，通过一个单独的docker-compose.yml模板文件，可以定义一组关联的应用容器为一个项目

compose重要的两个概念

服务（service）：一个应用的容器，可以包含多个相同镜像运行的容器 项目（project）：由一组关联的应用容器组成的完整单元,在docker-compose.yml中定义

## 安装及使用

通过**包管理器**下载或者到**github**上下载

## 命令说明

compose的命令对象大部分是项目本身，也可以指定项目中的服务或容器，默认情况下都是项目

执行`docker-compose [cammand] --help`查看具体某个命令的使用

## compose模块文件

```yml
#compose版本
version: "3"
#定义服务
services:
  #服务名称
  webapp:
    #镜像名称
    image: examples/web
    #暴露的端口
    ports:
      - "80:80"
    #映射卷
    volumes:
      - "/data"
```

也可以通过build指定dockerfile文件自动构建镜像

```yaml
version: '3'
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
      cache_from:
        - alpine:latest
        - corp/web_app:3.14
```

- `args`:指定构建时的缓存，`cache_from`:指定构建时的缓存


- `cammand`:指定容器启动后执行的命令

## docker-omcpose常用命令
- build       Build or rebuild services

- convert     Converts the compose file to platform's canonical format

- cp          Copy files/folders between a service container and the local filesystem

- create      Creates containers for a service.

- down        Stop and remove containers, networks

- events      Receive real time events from containers.

- exec        Execute a command in a running container.

- images      List images used by the created containers

- kill        Force stop service containers.

- logs        View output from containers

- ls          List running compose projects

- pause       Pause services

- port        Print the public port for a port binding.

- ps          List containers

- pull        Pull service images

- push        Push service images

- restart     Restart containers

- rm          Removes stopped service containers

- run         Run a one-off command on a service.

- start       Start services

- stop        Stop services

- top         Display the running processes

- unpause     Unpause services

- up          Create and start containers

- version     Show the Docker Compose version information


>用法和docker命令一样



### 模板文件简介
ompose目前有三个版本分别为Version 1，Version 2，Version 3，Compose区分Version 1和Version 2（Compose 1.6.0+，Docker Engine 1.10.0+）。Version 2支持更多的指令。Version 1将来会被弃用

#### image
image是指定服务的镜像名称或镜像ID。如果镜像在本地不存在，Compose将会尝试拉取镜像。


#### build
服务除了可以基于指定的镜像，还可以基于一份Dockerfile，在使用up启动时执行构建任务，构建标签是build，可以指定Dockerfile所在文件夹的路径。Compose将会利用Dockerfile自动构建镜像，然后使用镜像启动服务容器。


##### context
context选项可以是Dockerfile的文件路径，也可以是到链接到git仓库的url，当提供的值是相对路径时，被解析为相对于撰写文件的路径，此目录也是发送到Docker守护进程的context

```yaml
build:
  context: ./dir
```

##### dockerfile
使用dockerfile文件来构建，必须指定构建路径

```yaml
build:
  context: .
  dockerfile: Dockerfile-alternate
```

#### command
启动命令

#### container_name
指定容器名称


#### depends_on
指定容器依赖


#### ports
ports用于映射端口的标签。　
使用HOST:CONTAINER格式或者只是指定容器的端口，宿主机会随机映射端口。

```yaml
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```

#### extra_hosts
添加主机名的标签，会在/etc/hosts文件中添加一些记录。

```yaml
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```

#### volumes
挂载一个目录或者一个已存在的数据卷容器，可以直接使用 [HOST:CONTAINER]格式，或者使用[HOST:CONTAINER:ro]格式，后者对于容器来说，数据卷是只读的，可以有效保护宿主机的文件系统。 Compose的数据卷指定路径可以是相对路径，使用 . 或者 .. 来指定相对目录。 数据卷的格式可以是下面多种形式


```yaml
volumes:
  // 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）。
  - /var/lib/mysql
  // 使用绝对路径挂载数据卷
  - /opt/data:/var/lib/mysql
  // 以 Compose 配置文件为中心的相对路径作为数据卷挂载到容器。
  - ./cache:/tmp/cache
  // 使用用户的相对路径（~/ 表示的目录是 /home/<用户目录>/ 或者 /root/）。
  - ~/configs:/etc/configs/:ro
  // 已经存在的命名的数据卷。
  - datavolume:/var/lib/mysql
```


#### expose
暴露端口，但不映射到宿主机，只允许能被连接的服务访问。仅可以指定内部端口为参数，如下所示：

```yaml
expose:
    - "3000"
    - "8000"
```

#### links
链接到其它服务中的容器。使用服务名称（同时作为别名），或者服务名称:服务别名（如 SERVICE:ALIAS），例如

```yaml
links:
    - db
    - db:database
    - redis
```


#### net
设置网络模式。

```yaml
net: "bridge"
net: "none"
net: "host"
```

#### dns
自定义DNS服务器。可以是一个值，也可以是一个列表。

```yaml
dns：8.8.8.8
dns：
    - 8.8.8.8    
    - 9.9.9.9
```
