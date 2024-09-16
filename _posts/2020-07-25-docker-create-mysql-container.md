---
title: 【Docker】创建MySQL容器
date: 2020-07-25 22:53 +0800
categories: [Docker]
tags: [docker, mysql]
---
## 参考
<https://hub.docker.com/_/mysql>

## 获取镜像

```shell
docker pull mysql
```

## 创建容器

```shell
docker run --name my-mysql -e MYSQL_ROOT_PASSWORD=my-passwd -d -p 3306:3306 mysql
```

* `--name`选项指定容器名字为my-mysql
* `-e`选项设置环境变量，MYSQL_ROOT_PASSWORD环境变量指定MySQL root用户的密码为my-passwd
* `-d`选项为后台运行
* `-p`选项指定端口映射，将容器的3306端口（冒号后）映射到主机的3306端口（冒号前）（注意：创建容器后无法通过命令修改）
* 最后的mysql为要使用的镜像名字，也可以指定版本标签，如mysql:latest或mysql:8.0.15

## 进入容器并连接到MySQL

```shell
docker exec -it my-mysql bash
mysql -uroot -pmy-passwd
```

使用主机的IP地址即可从外部连接，假设主机IP地址为192.168.1.1

```shell
mysql -h192.168.1.1 -uroot -pmy-passwd
```

注意：需要给容器设置端口映射才能从外部连接，如果是阿里云服务器还需要添加安全组规则

![阿里云服务器安全组规则](/assets/images/docker-create-mysql-container/阿里云服务器安全组规则.png)

## 数据存储位置
默认情况下，Docker自动管理MySQL的数据存储位置（将容器中的/var/lib/mysql 目录映射到主机磁盘上的某个位置/var/lib/docker/volumes/.../_data，可通过docker inspect my-mysql输出信息中的Mounts->Source查看，另见docker volume命令）。

要想自定义数据存储位置，需要在创建容器时指定`--mount`选项：

```shell
--mount type=bind,src=/my/own/datadir,dst=/var/lib/mysql
```
