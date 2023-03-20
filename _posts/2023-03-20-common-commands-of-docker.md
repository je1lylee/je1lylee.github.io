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