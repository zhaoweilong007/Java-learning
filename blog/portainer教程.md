<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Portainer管理docker镜像容器](#portainer%E7%AE%A1%E7%90%86docker%E9%95%9C%E5%83%8F%E5%AE%B9%E5%99%A8)
  - [部署](#%E9%83%A8%E7%BD%B2)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Portainer管理docker镜像容器

> docker的一款GUI的管理工具，支持对docker进行启动、停止、部署、删除等等操作，界面美观，功能强大

## 部署

部署非常简单，拉取portainer镜像，运行即可，[docker hub文档](https://hub.docker.com/r/portainer/portainer)

- linux部署

~~~shell
docker pull portainer/portainer
~~~

~~~shell
docker volume create portainer_data
~~~

~~~shell
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
~~~

window部署请看[window](https://documentation.portainer.io/v2.0/deploy/ceinstalldocker/)

- 启动成功访问 http://localhost:9000 出现管理界面，第一次需要你设置密码

![portainer](../images/portainer.png)

管理docker容器
![portainer2](../images/portainer2.png)

管理docker镜像
![portainer-iamges](../images/portainer-images.png)