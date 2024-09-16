---
title: 【Docker】创建JanusGraph容器
date: 2020-07-27 11:13 +0800
categories: [Docker]
tags: [docker, janusgraph]
---
## 参考
* <https://hub.docker.com/r/janusgraph/janusgraph>
* <https://docs.janusgraph.org/getting-started/installation/>
* <https://www.jianshu.com/p/c39f6ebaf9b5>

## 获取镜像

```shell
docker pull janusgraph/janusgraph
```

## 创建容器

```shell
docker run --name janusgraph-default -d -p 8182:8182 janusgraph/janusgraph
```

## 进入容器

```shell
docker exec -it janusgraph-default bash
```

![进入容器](/assets/images/docker-create-janusgraph-container/进入容器.png)

![目录结构](/assets/images/docker-create-janusgraph-container/目录结构.png)

可以看到默认进入的工作目录为容器的/opt/janusgraph目录，和下载的JanusGraph客户端解压后的目录结构相同，bin目录包含服务器和客户端的启动脚本，conf目录包含配置文件。

在容器中执行ps命令查看进程，可以看到在启动容器时自动执行了`bash bin/gremlin-server.sh /etc/opt/janusgraph/gremlin-server.yaml`命令来启动Gremlin服务

![查看进程](/assets/images/docker-create-janusgraph-container/查看进程.png)

## 连接到数据库
[JanusGraph使用教程]({% post_url 2019-07-23-janusgraph-tutorial %})“连接远程Gremlin服务器”

在容器内部连接：直接执行bin/gremlin.sh脚本打开Gremlin控制台，之后执行

```shell
:remote connect tinkerpop.server conf/remote.yaml
:remote console
```

![在容器内部连接](/assets/images/docker-create-janusgraph-container/在容器内部连接.png)

在外部连接：先将conf/remote.yaml中的hosts修改为容器所在主机的IP地址

![修改hosts配置](/assets/images/docker-create-janusgraph-container/修改hosts配置.png)

如果是阿里云服务器则需要添加安全组规则

![阿里云服务器安全组规则](/assets/images/docker-create-janusgraph-container/阿里云服务器安全组规则.png)

之后用同样的方法连接远程服务器
  
![在外部连接](/assets/images/docker-create-janusgraph-container/在外部连接.png)
