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

官方提供了几种程序包，我们使用`Gateway Server Binary`即可。具体的下载以及解压参考[Knox 1.6.0 User Guide](https://knox.apache.org/books/knox-1-6-0/user-guide.html)。



# 部署时的坑

## HTTPS

Knox是支持https的，但是需要证书。这里我们并不需要配置证书，这一点在刚开始的时候并没有注意到导致浪费了非常多的时间，将下面的配置加入`gateway-site.xml`中。

```xml
<!-- 关掉HTTPS -->
<property>
        <name>ssl.enabled</name>
        <value>false</value>
    </property>
 
<!-- 开放所有请求 -->
    <property>
        <name>gateway.dispatch.whitelist</name>
        <value>^.*$</value>
        <description>The whitelist to be applied for dispatches associated with the service roles specified by gateway.dispatch.whitelist.services.
        If the value is DEFAULT, a domain-based whitelist will be derived from the Knox host.</description>
    </property>
```

## authentication

knox是提供了多种的身份验证方式，比如默认的sandbox使用的就是这种方式。

```xml
<provider>
            <role>authentication</role>
            <name>ShiroProvider</name>
            <enabled>true</enabled>
            <param>
                <!--
                session timeout in minutes,  this is really idle timeout,
                defaults to 30mins, if the property value is not defined,,
                current client authentication would expire if client idles contiuosly for more than this value
                -->
                <name>sessionTimeout</name>
                <value>30</value>
            </param>
            <param>
                <name>main.ldapRealm</name>
                <value>org.apache.knox.gateway.shirorealm.KnoxLdapRealm</value>
            </param>
            <param>
                <name>main.ldapContextFactory</name>
                <value>org.apache.knox.gateway.shirorealm.KnoxLdapContextFactory</value>
            </param>
            <param>
                <name>main.ldapRealm.contextFactory</name>
                <value>$ldapContextFactory</value>
            </param>
            <param>
                <name>main.ldapRealm.userDnTemplate</name>
                <value>uid={0},ou=people,dc=hadoop,dc=apache,dc=org</value>
            </param>
            <param>
                <name>main.ldapRealm.contextFactory.url</name>
                <value>ldap://localhost:33389</value>
            </param>
            <param>
                <name>main.ldapRealm.contextFactory.authenticationMechanism</name>
                <value>simple</value>
            </param>
            <param>
                <name>urls./**</name>
                <value>authcBasic</value>
            </param>
        </provider>
```

如果我们不需要任何身份校验，不能简单粗暴的采取如下方式，否则启动的时候将会抛出异常。

```xml
<provider>
            <role>authentication</role>
            <name>ShiroProvider</name>
            <enabled>false</enabled>
</provider>
```

而是应该采取`匿名验证`的方式，就像下面这样

```xml
<provider>
            <role>authentication</role>
            <name>Anonymous</name>
            <enabled>true</enabled>
        </provider>
```

# ending

Knox的能力远不止gateway，它可以与Kerberos进行紧密的耦合实现完整的Hadoop集群身份验证。或者是使用LDAP来进行简单的身份验证以及访问的审计功能。这里我们只用了gateway来解决眼下最着急的问题，终于又可以开心愉快的访问YARN了。