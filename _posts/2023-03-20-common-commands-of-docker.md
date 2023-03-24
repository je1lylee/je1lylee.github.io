---
title: docker的常用命令
author: jelly_lee
date: 2023-03-20 20:53:13 +0800
categories: [Docker]
tags: [docker, dockerfile]
---



最近跟docker打交道比较多，正好记录一下docker及dockerfile常用的命令及语法。



# 通过Dockerfile定义一个镜像

想要运行一个容器首先需要一个镜像，这里我们通过Dockerfile创建一个镜像。下面来介绍一下常用的命令。



## FROM

通过FROM可以定义你的镜像将基于那个镜像进行创建。

## RUN

通过RUN命令你可以执行一些命令，就和在shell里执行一样，比如yum/wget/tar/chmod等。

## ADD

将当前路径下的某个文件或某些文件加入到容器内，比如在项目流水线的dockerfile中经常看到`ADD . .`这种神奇的操作。另外它在添加tar文件时会自动解压，它也可以访问网络资源并将其添加到容器内。

## COPY

和ADD类似，但是它只能copy，不会对tar文件进行解压，也不能访问网络资源。

## WORKDIR

定义当前的工作目录，比如`WORKDIR /app`那么后续的操作都会在这个目录下进行。

## MAINTAINER

在dockerfile中留下维护者的名字，通常格式有下面几种:

- MAINTAINER Jelly Lee
- MAINTAINER je1lylee@163.com
- MAINTAINER Jelly Lee <je1lylee@163.com>

## LABEL

为镜像添加一些元数据。

## ENV

定义环境变量。

## EXPOSE

定义暴露出的端口，这里只是定义，如果在创建容器的时候没有将端口暴露出来，那么也是无法访问到这个容器的。

## CMD

在容器启动时执行命令，通常它会被当作`ENTRYPOINT`来使用。

## ENTRYPOINT

这个是正主，他们的区别是docker run命令中指定的任何参数都会作为参数传递给entrypoint，dockerfile中只能有一个entrypoint命令，如果多次制定那么只会执行最后一个。



下面是一个简单的dockerfile例子

```dockerfile
FROM centos
MAINTAINER Jelly Lee <je1lylee@163.com>
WORKDIR /app
ADD . .
EXPOSE 8080
ENV ENV=online
ENTRYPOINT ["sh","-c","/app/docker-entrypoint.sh ${ENV}"]
```

它表达的意思是要使用centos作为基础镜像，/app作为工作目录，暴露8080端口并将ENV作为参数传给一个叫docker-entrypoint.sh的脚本。

# 构建镜像

现在我们有定义镜像的dockerfile了，接下来就是构建它并发布了。构建一个镜像非常简单，就像下面这样：

```shell
image=harbor-registry.corp.org/app/demo:1.0
docker build -t $image .
```

第一行我们定义了一个镜像的tag，表达的是在app空间下一个叫demo的镜像，tag是1.0。第二行则是将当前目录下的dockerfile进行构建。

执行下面的命令即可将构建出的镜像推送到远程仓库，当然大部分镜像仓库对于写操作都是要求登录的，执行`docker login <repo>` 即可进行登录，默认的repo是`Docker Hub`这里可以根据情况自由选择。

```shell
docker push $image
```

# 运行一个容器

运行容器非常简单，比如这样：

```shell
[root@ligd-laptop ~]# docker run -it centos:centos7
Unable to find image 'centos:centos7' locally
centos7: Pulling from library/centos
2d473b07cdd5: Already exists
Digest: sha256:be65f488b7764ad3638f236b7b515b3678369a5124c47b8d32916d6487418ea4
Status: Downloaded newer image for centos:centos7
[root@e0828e112c65 /]#
```

这样你就可以获得一个centos的容器环境。现在我们该玩儿点有趣的了，比如暴露一个web端口。这里我我们通过端口映射的方式把宿主机的8980映射到容器的8080端口。

```shell
[root@ligd-laptop ~]# docker run -itd -p 8980:8080 centos:centos7
da32d849dc57c0f90861d617587436d7b951378d0a923a643fedaae48fd6741d
```

我们这是做了映射，并没有在容器内启动任何监听8080的服务，这里使用`netstat`来看看是不是映射上了。

```shell
[root@ligd-laptop ~]# netstat -antlp | grep 8980
tcp        0      0 0.0.0.0:8980            0.0.0.0:*               LISTEN      21326/docker-proxy
tcp6       0      0 :::8980                 :::*                    LISTEN      21333/docker-proxy
```

映射完网络，接下来该挂载驱动器(volume)了,比如这里我想把`/opt/software/grafana-9.3.2`映射到容器内的`/opt/`目录下。

```shell
[root@ligd-laptop grafana-9.3.2]# docker run -it -v /opt/software/grafana-9.3.2:/opt centos:centos7
[root@2717b2ce6b67 /]# ls /opt/
LICENSE          README.md        bin/             data/            public/
NOTICE.md        VERSION          conf/            plugins-bundled/ scripts/
```

这样我们在容器内操作这个目录实际上是在操作宿主机上的目录。

