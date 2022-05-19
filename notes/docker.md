<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Docker](#docker)
  - [安装](#%E5%AE%89%E8%A3%85)
    - [windows](#windows)
    - [ubuntu20.04](#ubuntu2004)
  - [常用命令](#%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4)
    - [镜像相关](#%E9%95%9C%E5%83%8F%E7%9B%B8%E5%85%B3)
    - [容器相关](#%E5%AE%B9%E5%99%A8%E7%9B%B8%E5%85%B3)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Docker
> Docker 是一个开源项目，诞生于 2013 年初，最初是 dotCloud 公司内部的一个业余项目。它基于 Google 公司推出的 Go 语言实现。 项目后来加入了 Linux 基金会，遵从了 Apache 2.0 协议，项目代码在 GitHub 上进行维护。

> Docker 自开源后受到广泛的关注和讨论，以至于 dotCloud 公司后来都改名为 Docker Inc。Redhat 已经在其 RHEL6.5 中集中支持 Docker；Google 也在其 PaaS 产品中广泛应用。

> Docker 项目的目标是实现轻量级的操作系统虚拟化解决方案。 Docker 的基础是 Linux 容器（LXC）等技术。

> 在 LXC 的基础上 Docker 进行了进一步的封装，让用户不需要去关心容器的管理，使得操作更为简便。用户操作 Docker 的容器就像操作一个快速轻量级的虚拟机一样简单


## 安装

### windows

下载[docker for windows](https://www.docker.com/products/docker-desktop)安装,


### ubuntu20.04


安装

- 更新软件依赖，`sudo apt update`
- 安装必要组件，`sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common`
- 导入源仓库的 GPG key,`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
- 姜docker apt加入源，`sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`
- 更新源，`sudo apt update`
- 安装docker，`sudo apt install docker-ce docker-ce-cli containerd.io`



## 常用命令

### 镜像相关

**查看镜像**

**`docker images`**

-a，–all=false，显示所有镜像

-f，–filter=[]，显示时过滤条件

–no-trunc=false，指定不使用截断的形式显示数据

-q，–quiet=false，只显示镜像的唯一id

**搜索镜像**

**`docker search 镜像名称:版本号`**

–automated=false，仅显示自动化构建的镜像

–no-trunc=false，不以截断的方式输出

–filter，添加过滤条件

**拉取镜像**

**`docker pull 镜像名称:版本号`**

-a，–all-tags=false，下载所有的镜像（包含所有TAG）



**推送镜像**

**`docker push 镜像名称:版本号`**

**标记镜像**

**`docker tag 原镜像 新镜像`**

docker tag <existing-image> <hub-user>/<repo-name>[:<tag>]


**移除镜像**

**`docker rmi 镜像id`**


### 容器相关


**创建容器**

**docker create [OPTIONS] IMAGE [COMMAND] [ARG...]**

创建一个新容器但不启动它


**运行容器**

**docker run IMAGE_NAME [COMMAND] [ARG…]**

如果镜像不存在会先下载镜像
如：
```bash
docker run hello-world
```

交互式的方式运行容器
- docker run -t -i –name=自定义名称 IMAGE_NAME /bin/bash

**参数说明**

-i –interactive=true | false，默认是false

-t –tty=true | false，默认是false

-d 启动守护式容器

–name 给启动的容器自定义名称，方便后续的容器选择操作

-P，–publish-all=true | false，大写的P表示为容器暴露的所有端口进行映射；

-p，–publish=[]，小写的p表示为容器指定的端口进行映射，有四种形式：

- containerPort：只指定容器的端口，宿主机端口随机映射；

- hostPort:containerPort：同时指定容器与宿主机端口一一映射；

- ip::containerPort：指定ip和容器的端口；

- ip:hostPort:containerPort：指定ip、宿主机端口以及容器端口。

**列出容器**
**docker ps [-a] [-l]**

-a all 列出所有容器

-l latest 列出最近的容器


**查看指定容器**

**docker inspect name | id**


**重新启动停止的容器**

**docker start name|id**

**重启容器**

**docker restart name|id**

**停止容器**

**docker stop name|id**

**删除容器**

**docker remove name|id**

**查看容器日志**

**docker logs [-f] [-t] [–tail] IMAGE_NAME**

-f –follows=true | false，默认是false，显示更新

-t –timestamps=true | false，默认是false，显示时间戳

–tail=“all” | 行数，显示最新行数的日志


**查看容器内进程**

**docker top name|id**


**运行中容器启动新进程**

**docker exec [-d] [-i] [-t] IMAGE_NAME [COMMAND] [ARG…]**

```bash
docker exec -it ubuntu /bin/bash
```


  