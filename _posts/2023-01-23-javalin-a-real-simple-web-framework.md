---
title: Javalin Web原来如此简单
author: jelly_lee
date: 2023-01-23 17:02:13 +0800
categories: [Java]
tags: [java, kotlin, web, javalin]
---

有一次完成需求的时候需要暴露出一个web端口来接收http请求并完成一些功能，引入spring-web有点大材小用，所以在GitHub逛了一圈儿发现了javalin，需求搞完了，该研究研究这个玩意儿了。

# 介四嘛？

javelin的[官网](https://javalin.io/)，是这样描述它的

> A simple web framework for Java and Kotlin

看起来简单易用就是它最大的特性了，我们来搞一个简单的maven项目来引入javalin。引入如下坐标即可。

```xml
<dependency>
    <groupId>io.javalin</groupId>
    <artifactId>javalin</artifactId>
    <version>5.3.2</version>
</dependency>
```

我们来执行`mvn dependency:tree`来看看它都依赖了什么。

```
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------< com.github.je1lylee:javalin-demo >------------------
[INFO] Building javalin-demo 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ javalin-demo ---
[INFO] com.github.je1lylee:javalin-demo:jar:1.0-SNAPSHOT
[INFO] \- io.javalin:javalin:jar:5.3.2:compile
[INFO]    +- org.slf4j:slf4j-api:jar:2.0.6:compile
[INFO]    +- org.eclipse.jetty:jetty-server:jar:11.0.13:compile
[INFO]    |  +- org.eclipse.jetty.toolchain:jetty-jakarta-servlet-api:jar:5.0.2:compile
[INFO]    |  +- org.eclipse.jetty:jetty-http:jar:11.0.13:compile
[INFO]    |  |  \- org.eclipse.jetty:jetty-util:jar:11.0.13:compile
[INFO]    |  \- org.eclipse.jetty:jetty-io:jar:11.0.13:compile
[INFO]    +- org.eclipse.jetty:jetty-webapp:jar:11.0.13:compile
[INFO]    |  +- org.eclipse.jetty:jetty-servlet:jar:11.0.13:compile
[INFO]    |  |  \- org.eclipse.jetty:jetty-security:jar:11.0.13:compile
[INFO]    |  \- org.eclipse.jetty:jetty-xml:jar:11.0.13:compile
[INFO]    +- org.eclipse.jetty.websocket:websocket-jetty-server:jar:11.0.13:compile
[INFO]    |  +- org.eclipse.jetty.websocket:websocket-jetty-common:jar:11.0.13:compile
[INFO]    |  |  \- org.eclipse.jetty.websocket:websocket-core-common:jar:11.0.13:compile
[INFO]    |  +- org.eclipse.jetty.websocket:websocket-servlet:jar:11.0.13:compile
[INFO]    |  |  \- org.eclipse.jetty.websocket:websocket-core-server:jar:11.0.13:compile
[INFO]    |  \- org.eclipse.jetty:jetty-annotations:jar:11.0.13:compile
[INFO]    |     +- org.eclipse.jetty:jetty-plus:jar:11.0.13:compile
[INFO]    |     |  +- jakarta.transaction:jakarta.transaction-api:jar:2.0.0:compile
[INFO]    |     |  \- org.eclipse.jetty:jetty-jndi:jar:11.0.13:compile
[INFO]    |     +- jakarta.annotation:jakarta.annotation-api:jar:2.1.1:compile
[INFO]    |     +- org.ow2.asm:asm:jar:9.4:compile
[INFO]    |     \- org.ow2.asm:asm-commons:jar:9.4:compile
[INFO]    |        \- org.ow2.asm:asm-tree:jar:9.4:compile
[INFO]    +- org.eclipse.jetty.websocket:websocket-jetty-api:jar:11.0.13:compile
[INFO]    \- org.jetbrains.kotlin:kotlin-stdlib-jdk8:jar:1.7.10:compile
[INFO]       +- org.jetbrains.kotlin:kotlin-stdlib:jar:1.7.10:compile
[INFO]       |  +- org.jetbrains.kotlin:kotlin-stdlib-common:jar:1.7.10:compile
[INFO]       |  \- org.jetbrains:annotations:jar:13.0:compile
[INFO]       \- org.jetbrains.kotlin:kotlin-stdlib-jdk7:jar:1.7.10:compile
```

日志框架、Jetty和kotlin，当然官网也提到了javalin是基于jetty编写的，代码量也只是几千行。

>## Simple
>
>Unlike other Java and Kotlin web frameworks, Javalin has very few concepts that you need to learn. You never extend classes and you rarely implement interfaces.
>
>## Lightweight
>
>Javalin is just a few thousand lines of code on top of Jetty, and its performance is equivalent to raw Jetty code. Due to its size, it's very easy to reason about the source code.
>
>## Interoperable
>
>Other Java and Kotlin web frameworks usually offer one version for each language. Javalin is being made with inter-operability in mind, apps are built the same way in both Java and Kotlin.
>
>## Flexible
>
>Javalin is designed to be simple and blocking, as this is the easiest programming model to reason about. But, if you set a `Future` as a result, Javalin switches into asynchronous mode.
>
>## Educational
>
>Visit our [educators page](https://javalin.io/for-educators) if you're teaching web programming and looking for a web framework which will get out of your way and let you focus on the core concepts of your curriculum.
>
>## OpenAPI
>
>Many lightweight Java and Kotlin web frameworks don't support OpenAPI, but Javalin does (including Swagger UI and ReDoc). Learn more at the [OpenAPI plugin page](https://javalin.io/plugins/openapi).
>
>## Jetty
>
>Javalin runs on top of Jetty, one of the most used and stable web-servers on the JVM. You can configure the Jetty server fully, including SSL and HTTP3 and everything else that Jetty offers.

# 跑起来试试

让javelin跑起来非常简单，定义一个handler并提供一个端口即可，比如我们这儿定义访问根就返回一行文字，并选择在7070端口暴露这个http服务器。

```java
public class Main {
    public static void main(String[] args) {
        var app = Javalin.create()
                .get("/", ctx -> ctx.result("Hello World"))
                .start(7070);
    }
}
```

启动时在控制台发现了一些报错，因为`slf4j`是日志门面，需要和其他日志框架配合食用，这里我们按javalin的建议添加一个依赖。

```
SLF4J: No SLF4J providers were found.
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See https://www.slf4j.org/codes.html#noProviders for further details.

#########################################################################
Javalin: It looks like you don't have a logger in your project.
The easiest way to fix this is to add 'slf4j-simple':

pom.xml:
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>2.0.3</version>
</dependency>

build.gradle or build.gradle.kts:
implementation("org.slf4j:slf4j-simple:2.0.3")

Visit https://javalin.io/documentation#logging if you need more help
#########################################################################

```

重启项目以后我们发现javelin能够正常的输出日志了，访问`http://localhost:7070`也可以返回一行文字，这样一个简单的http server就有了。

````
[main] INFO io.javalin.Javalin - Starting Javalin ...
[main] INFO org.eclipse.jetty.server.Server - jetty-11.0.13; built: 2022-12-07T20:47:15.149Z; git: a04bd1ccf844cf9bebc12129335d7493111cbff6; jvm 17.0.1+12-LTS-39
[main] INFO org.eclipse.jetty.server.session.DefaultSessionIdManager - Session workerName=node0
[main] INFO org.eclipse.jetty.server.handler.ContextHandler - Started i.j.j.@5ffead27{/,null,AVAILABLE}
[main] INFO org.eclipse.jetty.server.AbstractConnector - Started ServerConnector@548ad73b{HTTP/1.1, (http/1.1)}{0.0.0.0:7070}
[main] INFO org.eclipse.jetty.server.Server - Started Server@50de0926{STARTING}[11.0.13,sto=0] @245ms
[main] INFO io.javalin.Javalin - 
       __                  ___          ______
      / /___ __   ______ _/ (_)___     / ____/
 __  / / __ `/ | / / __ `/ / / __ \   /___ \
/ /_/ / /_/ /| |/ / /_/ / / / / / /  ____/ /
\____/\__,_/ |___/\__,_/_/_/_/ /_/  /_____/

       https://javalin.io/documentation

[main] INFO io.javalin.Javalin - Listening on http://localhost:7070/
[main] INFO io.javalin.Javalin - You are running Javalin 5.3.2 (released January 22, 2023).
[main] INFO io.javalin.Javalin - Javalin started in 133ms \o/
````

