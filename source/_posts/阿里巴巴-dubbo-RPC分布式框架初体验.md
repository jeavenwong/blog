---
title: 阿里巴巴 dubbo RPC分布式框架初体验
date: 2020-07-05 17:10:54
tags:
---

最近写了一个秒杀系统，主要使用 Spring Boot，用到了微服务的思想。之前简单学过丁雪丰老师的 Spring 全家桶课程，初步认识了 Spring Cloud Netflix OSS ，感觉使用起来非常顺滑，但是很多组件 Netflix 也已经不维护了。阿里在开源上是花了很大精力的，Dubbo 作为阿里比较早的开源项目，虽然之前一直被边缘化，但是随着 Spring Cloud Alibaba 的兴起，Dubbo 也和 Spring Cloud 结合的更加紧密。（PS: Dubbo 使用起来确实没有 Spring Cloud 一站式服务方便）

<!--more-->

Dubbo 阿里已经捐给了  Apache 基金会，作为顶级开源项目 Apache Dubbo 来维护。同时阿里还有自己维护的开源项目 Alibaba Dubbo，我使用的就是 Alibaba Dubbo。

##### 开发环境

- OS: Windows10
- IDE: Intellij IDEA 2018.2
- Build: Maven 3.3.9
- Framework: Spring Boot 2.0.1.RELEASE (内嵌Tomcat)
- 主要组件：ZooKeeper 3.4.6（注册中心），Dubbo 2.6.8，Dubbo Admin 2.6.x
- 调试工具: Postman 

##### 样例程序编写

程序提供应用和服务消费应用均采用内置的 Tomcat，直接在 debug 模式下以可执行 jar 包运行。

创建空的 maven 项目，项目的结构如下图所示，一个父模块 + 三个子模块（提供者模块、消费者模块、公共API模块）。

![structure](structure.JPG)

API模块的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>DemoDubbo</artifactId>
        <groupId>com.jeaven</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>ServiceInterface</artifactId>

    <packaging>jar</packaging>

    <dependencies>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
        </dependency>
    </dependencies>

</project>
```



API模块的接口

消费者接口

```java
package com.jeaven.api.service;

import com.jeaven.api.model.UserAddress;

import java.util.List;

/**
 * @author Jeaven Wong (Jianwei Wang)
 * @date 2020/7/4 23:01
 */

public interface OrderService {
    //初始化订单
    public List<UserAddress> initOrder(String userId);
}

```

服务者接口

```java
package com.jeaven.api.service;

import com.jeaven.api.model.UserAddress;

import java.util.List;

/**
 * @author Jeaven Wong (Jianwei Wang)
 * @date 2020/7/4 23:01
 */
public interface UserService {
    //按照用户id返回所有的收货地址
    public List<UserAddress> getUserAddressList(String userId);
}

```

API模块的数据

```java
package com.jeaven.api.model;

import lombok.*;

import java.io.Serializable;

/**
 * @author Jeaven Wong (Jianwei Wang)
 * @date 2020/7/4 22:57
 */
//
//将提供方和消费方共同使用的POJO对象及接口抽离出来，减少代码冗余，接口实现放在其各自的包中。
//（将用户服务和订单服务都会用到的地址类提取出来，同时也把定义了获取地址方法的接口单独提取出来，共同放在一个API包中，
//消费方和提供方通过引入该API包，实现该包中的接口来定义各自获取地址的方法实现）

@Data
@AllArgsConstructor
@NoArgsConstructor
@ToString
@Builder
public class UserAddress implements Serializable {
    private Integer id;
    private String userAddress; //用户地址
    private String userId; //用户id
    private String consignee; //收货人
    private String phoneNum; //电话号码
    private String isDefault; //是否为默认地址    Y-是     N-否
}

```



服务提供模块

依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>DemoDubbo</artifactId>
        <groupId>com.jeaven</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>Provider</artifactId>

    <packaging>jar</packaging>

    <dependencies>

        <!--引入api接口模块-->
        <dependency>
            <groupId>com.jeaven</groupId>
            <artifactId>ServiceInterface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!--引入springboot的相关依赖-->
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--引入dubbo的依赖-->
        <!-- https://mvnrepository.com/artifact/com.alibaba/dubbo -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.8</version>
        </dependency>


        <!--springboot测试依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- 添加一个依赖
        dubbo项目启动报错:
        java.lang.NoClassDefFoundError: io/netty/channel/nio/NioEventLoopGroup
        -->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.32.Final</version>
        </dependency>


        <!-- 添加一个依赖
       dubbo项目启动报错:
       java.lang.NoClassDefFoundError: org/apache/curator/framework/CuratorFrameworkFactory
       -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.13.0</version>
        </dependency>

        <!--为了使用telnet-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.4</version>
        </dependency>

    </dependencies>

    <build>
        <finalName>Provider</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

以默认配置方式配置dubbo，即 application.properties

```properties
#当前服务/应用的名字
dubbo.application.name=Provider

#注册中心的协议和地址
dubbo.registry.protocol=zookeeper
dubbo.registry.address=127.0.0.1:2181

#通信规则（通信协议和接口）
dubbo.protocol.name=dubbo
dubbo.protocol.port=20880

#连接监控中心
dubbo.monitor.protocol=registry

#开启包扫描，可替代 @EnableDubbo 注解
##dubbo.scan.base-packages=com.jeaven

#qos的配置
dubbo.application.qosEnable=true
dubbo.application.qosPort=22222
dubbo.application.qosAcceptForeignIp=true

#内置tomcat端口号
server.port=9992
```

配置 dubbo 的 log4j，即 log4j.properties

```properties
log4j.rootLogger=INFO, stdout, D

# Console Appender
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern= %d{hh:mm:ss,SSS} [%t] %-5p %c %x - %m%n

# Custom tweaks
log4j.logger.com.codahale.metrics=WARN
log4j.logger.com.ryantenney=WARN
log4j.logger.com.zaxxer=WARN
log4j.logger.org.apache=WARN
log4j.logger.org.hibernate=WARN
log4j.logger.org.hibernate.engine.internal=WARN
log4j.logger.org.hibernate.validator=WARN
log4j.logger.org.springframework=WARN
log4j.logger.org.springframework.web=WARN
log4j.logger.org.springframework.security=WARN

# log file
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.File =C://Users//jeave//Desktop//test//provider.log
log4j.appender.D.Append = true
log4j.appender.D.Threshold = DEBUG 
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n
```

```java
package com.jeaven.provider.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

/**
 * @author Jeaven Wong (Jianwei Wang)
 * @date 2020/7/5 14:54
 */

@PropertySource("classpath:log4j.properties")
@Configuration
public class LoggingConfig {

}

```

UserService 接口实现

```java
package com.jeaven.provider.service;

import com.alibaba.dubbo.config.annotation.Service;
import com.jeaven.api.model.UserAddress;
import com.jeaven.api.service.UserService;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

/**
 * @author Jeaven Wong (Jianwei Wang)
 * @date 2020/7/4 22:39
 */

@Component
@Service(timeout = 3000)  //这个是dubbo的注解，用来暴露服务
public class UserServiceImpl implements UserService {
//    测试逻辑
    @Override
    public List<UserAddress> getUserAddressList(String userId) {
        UserAddress userAddress = new UserAddress().builder().userId(userId).userAddress("湖北襄阳")
                .id(0).consignee("jeavenwong").phoneNum("188XXXXXXXX")
                .isDefault("Y").build();
        List<UserAddress> resList = new ArrayList<>();
        resList.add(userAddress);
        return resList;
    }
}

```

主程序

```java
package com.jeaven.provider;

import com.alibaba.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author Jeaven Wong (Jianwei Wang)
 * @date 2020/7/4 22:44
 */

@SpringBootApplication
@EnableDubbo // 开启基于注解的dubbo功能（主要是包扫描@DubboComponentScan）,也可以在配置文件中使用dubbo.scan.base-package来替代   @EnableDubbo
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}

```

服务消费模块

依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>DemoDubbo</artifactId>
        <groupId>com.jeaven</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>Consumer</artifactId>

    <packaging>jar</packaging>

    <dependencies>

        <!--引入api接口模块-->
        <dependency>
            <groupId>com.jeaven</groupId>
            <artifactId>ServiceInterface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!--引入springboot的相关依赖-->
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <!--引入dubbo的依赖-->
        <!-- https://mvnrepository.com/artifact/com.alibaba/dubbo -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.8</version>
        </dependency>


        <!--springboot测试依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.32.Final</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.13.0</version>
        </dependency>

        <!--为了使用telnet-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.4</version>
        </dependency>

    </dependencies>

    <build>
        <finalName>Consumer</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

同理，配置 dubbo

```properties
#避免和监控中心端口冲突，设为 tomcat 8081端口访问
server.port=9991

#下面是dubbo的配置
dubbo.application.name=Consumer
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.monitor.protocol=registry

#qos的配置
#dubbo.application.qos.enable=true
#dubbo.application.qos.port=33333
#dubbo.application.qos.accept.foreign.ip=true

dubbo.application.qosEnable=true
dubbo.application.qosPort=33333
dubbo.application.qosAcceptForeignIp=true
```

同理，配置日志

```properties
log4j.rootLogger=INFO, stdout, D

# Console Appender
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern= %d{hh:mm:ss,SSS} [%t] %-5p %c %x - %m%n

# Custom tweaks
log4j.logger.com.codahale.metrics=WARN
log4j.logger.com.ryantenney=WARN
log4j.logger.com.zaxxer=WARN
log4j.logger.org.apache=WARN
log4j.logger.org.hibernate=WARN
log4j.logger.org.hibernate.engine.internal=WARN
log4j.logger.org.hibernate.validator=WARN
log4j.logger.org.springframework=WARN
log4j.logger.org.springframework.web=WARN
log4j.logger.org.springframework.security=WARN

# log file
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.File =C://Users//jeave//Desktop//test//consumer.log
log4j.appender.D.Append = true
log4j.appender.D.Threshold = DEBUG 
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n
```

测试用的 Controller

```java
package com.jeaven.consumer.controller;

import com.jeaven.api.model.UserAddress;
import com.jeaven.api.service.OrderService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * @author Jeaven Wong (Jianwei Wang)
 * @date 2020/7/5 8:08
 */

@Slf4j
@RestController
public class MyController {

    @Autowired
    private OrderService orderService;

    @RequestMapping(value = "/initOrder", method = RequestMethod.GET)
    public List<UserAddress> getService(@RequestParam("uid") String uid) {
        List<UserAddress> res = orderService.initOrder(uid);
        log.info("返回的数据是: {}", res);
        return res;
    }

    @RequestMapping(value = "/test", method = RequestMethod.GET)
    public String test() {
        log.info("hello.wrold.....");
        return "hello.wrold...";
    }
}

```

实现 OrderService 接口

```java
package com.jeaven.consumer.service;

import com.alibaba.dubbo.config.annotation.Reference;
import com.jeaven.api.model.UserAddress;
import com.jeaven.api.service.OrderService;
import com.jeaven.api.service.UserService;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @author Jeaven Wong (Jianwei Wang)
 * @date 2020/7/5 8:47
 */

@Service  // 注意这个是 spring 的注解，不是 dubbo 的，搞错了这个 debug 了好久
public class OrderServiceImpl implements OrderService {

    @Reference
    private UserService userService;  // 调用远程服务

    @Override
    public List<UserAddress> initOrder(String userId) {
        return userService.getUserAddressList(userId);
//        UserAddress userAddress = new UserAddress().builder().isDefault("Y").phoneNum("188XXXXXXXX").consignee("jeavenwong")
//                .userId(userId).id(1).userAddress("hubei xiangyang").build();
//        List<UserAddress> list = new ArrayList<>();
//        list.add(userAddress);
//        return list;
    }
}

```

主程序

```java
package com.jeaven.consumer;

import com.alibaba.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author Jeaven Wong (Jianwei Wang)
 * @date 2020/7/4 23:33
 */

@SpringBootApplication
@EnableDubbo
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class,  args);
    }
}

```

之后，直接在 IDEA 里面使用 Maven clean package 进行编译，在主程序类上鼠标右键选择debug模式运行即可。无需再使用外置的 Servlet 容器。

##### 运行

运行的时候先开启 ZooKeeper， 双击 bin 目录下的Zserver.md 即可开启服务。

然后开启 Dubbo Admin 组件，我选择的是 2.6.x 的老版本，这个版本是使用 Spring Boot + Volecity 服务端模板写的，所以使用 IDEA  Maven 编译起来非常方便，而新的版本采用了前后分离的技术，前端使用 Vue.js，后端时Sping Boot，所以使用需要用 npm 编译前端，用 Maven 编译后端，易用性不高。

我 fork 的 Dubbo Admin 库地址是: https://gitee.com/jeavenwong/dubbo-admin

直接 git clone 到本地，然后修改配置文件再变异成可执行 jar 包运行即可。

可以参考博客: https://www.cnblogs.com/zjfjava/p/9694540.html

##### 运行结果

Postman get请求截图

![res1](res1.JPG)

Dubbo Admin 运行截图

![res2](res2.JPG)

因为服务提供者和服务消费者都开启了监控，所以显示了三个服务消费者

![res3](res3.JPG)

服务提供者只有一个

![res4](res4.JPG)

但是遗憾的是我没有运行成功 dubbo-monitor-simple ，不能在界面 UI 上显示出服务的调用次数和时间。

##### 参考

1. http://dubbo.apache.org/zh-cn/docs/user/configuration/annotation.html
2.  https://www.cnblogs.com/zjfjava/p/9696086.html#top
3. https://blog.csdn.net/csonst1017/article/details/100555291
4. https://www.cnblogs.com/zjfjava/p/9694540.html

##### 程序 Demo 地址

https://gitee.com/jeavenwong/DubboDemo







