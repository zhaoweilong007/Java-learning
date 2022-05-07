<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [使用harbor搭建docker私有仓库](#%E4%BD%BF%E7%94%A8harbor%E6%90%AD%E5%BB%BAdocker%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93)
  - [环境准备](#%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
  - [安装](#%E5%AE%89%E8%A3%85)
  - [开始上传镜像到harbor](#%E5%BC%80%E5%A7%8B%E4%B8%8A%E4%BC%A0%E9%95%9C%E5%83%8F%E5%88%B0harbor)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 使用harbor搭建docker私有仓库

> 一般公司都有自己docker私有仓库，而harbor就是其中之一，下面教你使用harbor搭建私有的docker仓库

## 环境准备

- Docker engine :Version 17.06.0-ce+ or higher
- Docker Compose : Version 1.18.0 or higher
- Openssl : Latest is preferred

## 安装

harbor可以在线安装和离线安装，不过方法都一样

- [下载harbor安装包](https://github.com/goharbor/harbor/releases) 在线或者离线都可以

使用`curl`下载

~~~shell
curl -LJO https://github.com/goharbor/harbor/releases/download/v2.2.1/harbor-online-installer-v2.2.1.tgz
~~~

- 解压harbor压缩包

~~~shell
tar xvf harbor-online-installer-version.tgz
~~~

或者

~~~shell
tar xvf harbor-offline-installer-version.tgz
~~~

- 进入harbor目录，赋值harbor.yml.tmp1的配置文件为harbor.yml

![harbor](../images/harbor.png)

- 修改配置文件，修改域名、端口、https证书配置、默认密码等

![harbor-yml](../images/harbor-yml.png)

- 以管理员的身份执行install.sh

~~~shell
sudo ./install.sh
~~~

我这里使用的是在线安装，执行时会下载镜像

![harbor-install](../images/harbor-install.png)

- 打开浏览器访问harbor，`http://域名:端口` 域名和端口为自己配置的，账号默认为admin，密码默认为Harbor12345

![harbor-home](../images/harbor-home.png)

可以看到harbor的管理界面了，到此harbor部署成功了！

![harbor-home](../images/harbor-home2.png)

## 开始上传镜像到harbor

!> 如果你使用的是http协议，那需要在docker的配置文件中增加所信任的http地址

在`"insecure-registries"`属性中，增加你的harbor地址，然后重启docker

如：
` "insecure-registries": ["http://域名:端口"] `

- 登陆到你的harbor

~~~cmd
docker login harbor地址
~~~

- 首先在harbor的管理界面新建一个项目

![harbor-home](../images/harbor1.png)

- 将镜像打包

~~~cmd
docker tag source:tag target:tag
~~~

- push镜像

![harbor2](../images/harbor2.png)

可以看到已经推送成功了

![harbor2](../images/harbor3.png)

其他配置可以详细查看[官方文档](https://goharbor.io/)