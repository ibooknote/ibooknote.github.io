# Spring in Aciton, 5th Edition 阅读笔记

## 1、内容大纲

### 1.1、Part 1 构建Spring应用的基础主题

* Chapter 1 介绍Spring和Spring Boot，以及如何初始化Spring项目
* Chapter 2 讨论使用Spring MVC构建应用的Web层
* Chapter 3 深入探讨Spring 应用的数据持久化
* Chapter 4 使用Spring Security进行安全验证
* Chapter 5 展示如何使用Spring Boot配置Spring应用

### 1.2、Part 2 Spring应用与其他应用集成

* Chapter 6 扩展讨论Chapter 2中的Spring MVC如何编写REST APIS
* Chapter 7 Spring应用如何消费（Consume）REST API
* Chapter 8 通过异步通信，实现Spring应用与JMS、RabbitMQ或Kafka收发消息
* Chapter 9 讨论声明式应用程序集成

### 1.3、Part 3 Spring对反应式（reactive）编程的支持

* Chapter 10 Reactor -- 反应式编程库
* Chapter 11 介绍 Spring WebFlex框架
* Chapter 12 反应式数据持久化、Spring Data 与 Cassandra和Mongo DB

### 1.4、Part 4 打破整体应用模型，介绍Spring Cloud和微服务开发

* Chapter 13 服务发现 Spring & Netflix's Eureka
* Chapter 14 由配置服务器集中管理多个微服务的配置
* Chapter 15 介绍circuit breaker pattern（Hystrix）

### 1.5、Part 5 应用发布

* Chapter 16 介绍Spring Boot Actuator（执行器）
* Chapter 17 使用Spring Boot Admin部署管理应用程序
* Chapter 18 讨论如何暴露和消费Spring beans/JMX MBeans
* Chapter 19 如何发布Spring应用

## 2、笔记

重点问题：

* Spring是什么？
* Spring能做什么？
* Spring是如何做的？
* Spring Boot是什么？
* Spring Boot能做什么？
* Spring Boot项目如何初始化？
* Spring Boot项目如何开发、调试？
* Spring Boot项目如何发布、启动？
* Spring Boot项目如何更新程序？

### 2.1、Chapter 1

Spring Framework的新功能：

* microservices
* reactive programming
* Spring Boot

Spring是什么？

spring的核心是提供了一个容器，作为应用程序上下文环境，可以在其中创建和管理应用程序组件（beans）。

spring使用依赖注入（DI，dependency injection, a pattern)将组件捆绑在一起。

基于Java @Configuration和 @Bean注解的配置方式比基于XML方式有更安全的类型检查，有更高的可重构性。

自动配置：

* autowiring--自动注入组件依赖的bean。
* component scanning--自动扫描应用程序的classpath，发现组件。

Spring Boot 是Spring Framework的一个延伸，可以从几方面提升效率：

* *autoconfiguration--自动扫描、注入
	* classpath
	* environment
	* other factors

autoconfiguration不需要写代码和配置

#### 2.1.2、Spring应用程序初始化

Spring Initializr--Spring项目初始化工具

flesh out--填充

使用Spring Initializr的几种方法：

* 从Web应用 http://start.spring.io
* 从命令行使用 curl命令
* 从命令行使用Spring Boot命令行接口
* 使用Spring Tool Suite创建新项目时
* 使用IntelliJ IDEA创建新项目时
* 使用NetBeans创建新项目时

在使用Initializr初始化maven项目后，pom文件中的packaging配置的是JAR，因为JAR适合于发布到云服务，而WAR适合传统的Java应用服务器。

项目只需继承Spring Boot，指定Spring Boot的版本，由Spring Boot来管理其他依赖的库和兼容的版本。

pom文件最后由Spring Boot plugin结束，该插件的重要功能：

* 指定使用Maven运行应用
* 保证所有依赖包都包含在可执行的JAR文件中，并在运行时classpath中可见
* 在JAR文件中生成一个manifest文件，用于表示bootstrap class作为JAR包中的main class

