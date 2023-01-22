---
title: 部署Apache Knox的踩坑记录
author: jelly_lee
date: 2023-01-22 17:02:13 +0800
categories: [Hadoop, Knox]
tags: [hadoop, yarn, knox, gateway]
---



因为某些特殊的原因，运维不让直接访问YARN UI了。之前采取的方法是使用nginx代理来进行审计，蛋疼的是访问`Application Master`的时候不能正确的路由到正确的地址上，每次查问题的时候都要手动修正URL，多搞几次让人想捶桌摆烂。是时候该搞一个网关了！！

# yarn啊！我该怎么找到你！

想要实现正确的路由原理很简单，那就是简单粗暴的`rewrite`，可以简单的理解为把`host:port`替换为`url`，简单的举个例子，现在集群里的yarn-site.xml配置如下：

```xml
        <property>
                <name>yarn.resourcemanager.webapp.address.rm1</name>
                <value>rm1:8088</value>
        </property>
        <property>
                <name>yarn.resourcemanager.webapp.address.rm2</name>
                <value>rm2:8088</value>
        </property
```

那么访问对应的`webapp.address`即可访问YARN UI，当你配置了一个域名，比如`rm1.je1lylee.github.io`，访问YARN首页的时候一切都好，YARN会把你带到`rm1.je1lylee.github.io/cluster`，但是当你找到一个任务，打算访问`application master`的时候，你会发现它把你带到了`rm1:8088/proxy/application_xxxx_xxx/`这自然就不能访问了。这时我们就需要一个中间商帮我们把`rm1:8088`改写成`rm1.je1lylee.github.io`。

# why Knox

> ## REST API and Application Gateway for the Apache Hadoop Ecosystem
>
> The Apache Knox™ Gateway is an Application Gateway for interacting with the REST APIs and UIs
> of Apache Hadoop deployments.
>
> The Knox Gateway provides a single access point for all REST and HTTP interactions with Apache Hadoop
> clusters.

一个生态圈儿里的，肯定优先用一个生态圈儿里的东西啦～所以knox成为最佳选择。

# 搞一个试试

[official website](https://knox.apache.org/)

[releases](https://cwiki.apache.org/confluence/display/KNOX/Apache+Knox+Releases)

官方提供了几种程序包，我们使用`Gateway Server Binary`即可。具体的下载以及解压步骤就不bb了。



# 部署时的坑

## HTTPS

Knox是支持https的，但是需要证书。这里我们并不需要配置证书，这一点在刚开始的时候并没有注意到导致浪费了非常多的时间。