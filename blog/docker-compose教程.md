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

## 常用命令
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