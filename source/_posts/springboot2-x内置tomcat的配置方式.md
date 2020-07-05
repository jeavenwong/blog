---
title: springboot2.x内置tomcat的配置方式
date: 2020-07-05 11:10:09
tags:
---

因为开发过程中，使用本地的Tomcat，总是出现各种各样奇怪的错误。既然使用了Spring Boot 2.x，那打包成可执行 jar 包不香吗？当然，可执行 jar 包里面使用的是 sripng-boot-starter-web 起步依赖引入的众多依赖中的内嵌 Tomcat。

<!--more--> 

简单记录下两种配置 Spring Boot 内嵌 Tomcat 的方式。

#####  一、使用 springboot 默认配置 application.properties

![properties](properties.JPG)

application.properties 里面的配置是 springboot 默认加载的，只要在里面指定 tomcat 的端口号等配置即可。

##### 二、使用注解进行配置

![annotation](annotation.JPG)

如图在 config 目录下新建一个类对内嵌的 Tomcat 进配置。

##### 可执行 jar 包

直接使用 maven 进行打包，在classpath 目录下生成 jar 包。

```jvm
java -jar XXXX.jar
```

再使用命令行使用即可。

##### 题外话

springboot 2.x 把 tomcat 内置了，不需要外置的 servlet 容器，可以在程序中直接使用 debug 模式运行，默认是运行在内嵌的 tomcat 上。如果项目需要发布打包出去，打包成可执行 jar 包即可。如果需要使用外置的容器，需要排除内嵌的 tomcat 容器，可以参考教程: https://www.cnblogs.com/mbblog/p/12145958.html

