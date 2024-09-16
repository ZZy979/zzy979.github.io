---
title: Docker使用教程
date: 2020-07-25 11:32 +0800
categories: [Docker]
tags: [docker]
---
## 1.简介
Docker是一个开源的应用容器引擎，可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。
* 官方网站：<https://www.docker.com/>
* 官方文档：<https://docs.docker.com/>

## 2.基本概念
* 容器(container)：独立运行的一个或一组应用（个人理解：每个容器是一个小型的Linux系统，其中安装了某种软件，例如MySQL）
* 镜像(image)：用于创建容器的模板，镜像和容器的关系就像类和实例的关系

## 3.安装
在Ubuntu系统上安装Docker的方法：<https://docs.docker.com/engine/install/ubuntu/>

使用apt安装Docker引擎的方法

(1) 更新apt索引、安装依赖包

```shell
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

(2) 添加Docker官方GPG Key

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

(3) 设置稳定版仓库

```shell
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

(4) 安装Docker引擎

```shell
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

(5) 通过运行hello-world镜像验证安装成功

```shell
sudo docker run hello-world
```

## 4.常用命令
注意：以下命令均需要sudo，要想不使用sudo，需要把用户添加到docker组：

```shell
usermod -aG docker <username>
```

### 4.1 镜像管理
`docker pull NAME[:TAG]`  下载镜像，其中tag为版本号，默认为latest
* 注意："latest"仅代表该仓库当前的最新版本，**下载之后并不会自动更新**。要查看镜像的实际版本，使用`docker inspect`命令。例如下载了mysql:latest镜像后，执行`docker inspect mysql`命令，在输出信息的ContainerConfig->Env->MYSQL_VERSION可以看到实际版本
* Docker Hup: <https://hub.docker.com/>
* 使用阿里云镜像加速器：<https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors>

`docker images`  列出所有镜像

`docker search TERM`  搜索镜像

`docker rmi IMAGE`  删除镜像

`docker inspect NAME|ID`  输出Docker对象的详细信息，对象可以是镜像或容器

### 4.2 容器管理
`docker ps [OPTIONS]`  列出正在运行的容器
* `-a`  列出全部容器

`docker run [OPTIONS] IMAGE [COMMAND]`  在新容器中运行命令，相当于docker create+docker start
* `--name NAME`  指定容器的名字
* `-d`  后台运行
* `-e`  设置环境变量
* `-p [IP_ADDRESS:]HOST_PORT:CONTAINER_PORT`  指定主机和容器的端口映射

一般镜像都会有默认执行的命令，如MySQL镜像是启动MySQL服务

注意：创建容器后没有命令可以直接修改端口映射，但可以通过修改容器的配置文件实现，具体做法：
1. 停止容器：`docker stop my-container`
2. 停止Docker服务：`systemctl stop docker`
3. 修改/var/lib/docker/containers/\<容器ID\> 目录下的hostconfig.json和config.v2.json文件中的PortBindings
4. 启动Docker服务：`systemctl start docker`
5. 启动容器：`docker start my-container`

参考：
* <https://blog.csdn.net/m0_37886429/article/details/82757116>
* <https://stackoverflow.com/questions/19335444/how-do-i-assign-a-port-mapping-to-an-existing-docker-container>

`docker exec [OPTIONS] CONTAINER COMMAND`  在一个正在运行的容器中执行命令
* `-d`  后台运行
* `-i -t`  交互式操作

例如，进入容器并启动Bash：`docker exec -it my-container bash`

`docker create [OPTIONS] IMAGE`  创建一个容器，类似于`docker run -d`，只是没有启动容器
* `--name NAME`  指定容器的名字
* `-e VAR=value`  设置环境变量
* `-p [IP_ADDRESS:]HOST_PORT:CONTAINER_PORT`  指定主机和容器的端口映射

`docker start CONTAINER`  启动容器，其中CONTAINER可以是容器名字或id

`docker stop CONTAINER`  停止容器

`docker restart CONTAINER`  重新启动容器

`docker rm CONTAINER`  删除容器

`docker cp CONTAINER:SRC_PATH DEST_PATH`和`docker cp SRC_PATH CONTAINER:DEST_PATH`  在容器和主机之间拷贝文件/文件夹

例如：
* `docker cp my-container:/path/to/src/file.txt /path/to/dst`
将容器my-container中的文件/path/to/src/file.txt拷贝到主机的/path/to/dst目录(/path/to/dst/file.txt)
* `docker cp my-container:/path/to/src /path/to/dst`
将容器my-container中的目录/path/to/src拷贝到主机的/path/to/dst目录(/path/to/dst/src)
* `docker cp /path/to/src/file.txt my-container:/path/to/dst`
将主机的文件/path/to/src/file.txt拷贝到容器my-container中的/path/to/dst目录(/path/to/dst/file.txt)
* `docker cp /path/to/src my-container:/path/to/dst`
将主机的目录/path/to/src拷贝到容器my-container中的/path/to/dst目录(/path/to/dst/src)
