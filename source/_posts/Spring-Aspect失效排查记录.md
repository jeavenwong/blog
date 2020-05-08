---
title: Spring Aspect失效排查记录
date: 2020-03-29 12:39:40
tags: 
	- Spring
	- Aspect
---

本文记录一个低级错误导致 sping boot 切面失效的bug。

<!--more-->

在使用Spring Aspect拦截Controller某方法打印日志的时候，突然发现AOP失效了，拦截方法正常执行，但是定义的切面并没有正常执行，IDEA 里面没有提示任何错误信息，可以引用到被拦截的目标方法上去。仔细检查了一遍代码，发现切面定义的类不是Class，而是Aspect。

#### 原因：

我使用 maven 导入了AOP之后，在创建新的切面类的时候，点了创建Aspect。

![introduce_aop_support](introduce_aop_support.JPG)

![abnormal_create_aspect](abnormal_create_aspect.png)

然后创建的不是Class，而是Aspect...这就导致了代码没问题但是却无效的bug。

#### 反思：

在找到错误后，将 " Aspect " 改为 “ Class ”后，切面可以正常生效。

![spring_boot_introduce_aop_auto](spring_boot_introduce_aop_auto.JPG)

如上图所示，spring-boot-starter-aop 引入了一系列的依赖，这些依赖相当于是 IDEA 的对Aspect的增强插件，IDEA 可以在新建文件时提供一个直接创建 Aspect 的选项，但是不能用 javac 编译器进行编译，要使用 Ajc 编译器，需要在 IDEA compliler 设置里指定 Ajc 的文件位置。

![aspect_plugin_config](aspect_plugin_config.png)

指定 Ajc 编译器后按照 Aspect 的语法写好后编译运行。



至此，我把 " Aspect " 改为 " Class "就可以使切面生效了。

![normal_aspect](normal_aspect.JPG)



这里还需要注意的一点是：

spring boot 在使用 Aspect 类大的时候，如果使用 @Aspect 这样的 Java Config 的方式而不使用 XML 来配置切面的话，就需要额外的在切面类的加上 @component 注解，这个如果没有加上，也会导致spring boot找不到切面类的 bean。

加 @component 的原因官方解释如下：

*You may register aspect classes as regular beans in your Spring XML configuration, or autodetect them through classpath scanning - just like any other Spring-managed bean. However, note that the @Aspect annotation is not sufficient for autodetection in the classpath: For that purpose, you need to add a separate @Component annotation (or alternatively a custom stereotype annotation that qualifies, as per the rules of Spring’s component scanner).*

*您可以在Spring XML配置中注册aspect类，或者通过类路径扫描自动检测它们，就像任何其他Spring管理bean一样。但是，请注意，@aspect注释对于在类路径中自动检测是不够的：为了达到这个目的，您需要添加一个单独的@component注解（或者根据Spring的组件扫描器的规则来定义一个定制的原型注解）。*

