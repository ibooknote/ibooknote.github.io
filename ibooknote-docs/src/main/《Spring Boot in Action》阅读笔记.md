# 《Spring Boot in Action》阅读笔记


## 1. 启动Spring

内容概要：

* Spring Boot如何简化Spring应用开发
* Spring Boot的基本功能
* 配置一个Spring Boot工作空间（workspace）

**Spring Boot** 使开发者可以专注应用功能，从配置中解脱出来。其最重要一点就是将Spring从工作中剥离出来。

### 1.1 Spring rebooted

Spring开始时是作为JEE的轻量级替代，虽然在组件编码时是轻量级的，但在配置来说却是重量级的。

* 最初，Spring采用XML配置
* Spring 2.5开始引入基于注解的组件扫描
* Spring 3.0引入基于Java的配置，重构了XML选项

即使如此，仍然没有逃离配置。

像其他框架一样，Spring为你做了很多事，但你也需要做很多事作为回报。

项目依赖管理也是很麻烦的事情，需要考虑如何决定使用哪个库，以及哪个版本更合适。版本选择错误可能称为现实生产率的杀手。

Spring Boot改变了这一切。

#### 1.1.1 快速重新浏览一下Spring

Spring Web应用的最基本要素：

* 项目框架，通过Maven或Gradle构建的依赖关系，包括Spring MVC和Servlet API
* 一个web.xml文件，用于声明DispatcherServlet
* 一个Spring配置，用于启用Spring MVC
* 一个控制器类，用于响应HTTP请求
* 一个web应用服务器，例如Tomcat，用于部署应用程序

在Spring Boot下，只需要控制器就可以，不需要配置，不需要web.xml不需要构建说明，甚至不需要应用服务器。Spring Boot会处理应用执行的所有后勤工作。

#### 1.1.2 检查Spring Boot的基本要素

四个核心技术：

* 自动化配置 -- 自动为应用提供所需的配置
* Starter依赖 -- 告诉Spring Boot需要那些功能，它确保增加合适的库
* 命令行接口 -- 通过它，只需要编写应用代码，不需要额外的项目构建，就可以写出完整的应用
* 执行器（Actuator） -- 可以窥视运行中的Spring Boot应用的内部工作

每个功能都是用其方法简化Spring应用开发。

##### 自动配置

不需要为了依赖注入，而配置bean。只需要将需要的库放在应用程序的classpath中，Spring Boot就可以自动检测，并配置相应的依赖关系。

##### Starter依赖

* 需要哪个库？
* group和artifact是什么？
* 哪个版本合适？
* 是否会和项目中其他依赖很好的配合？

这些功能都由Spring Boot的starter依赖完成，它实际上是一个特殊的Maven（或Gradle）依赖，利用依赖传递。

库的合适版本由starters拉取，并结果整体测试，不需要担心兼容性问题。

##### 命令行接口（CLI）

CLI是Spring Boot的强大功能的一个可选项，可以简化Spring开发，但也可以不使用它。

##### 执行器

其他几个部分都是为了简化Spring开发，而Actuator的能力是在运行时检查应用程序的内部。通过Actuator，可以检查应用的内部工作：

* Spring应用环境中配置了那些bean？
* Spring Boot自动化配置做了哪些选择？
* 应用程序的环境变量、系统属性、配置属性、命令行参数由哪些？
* 应用程序线程的当前状态
* 追踪应用程序对最近的HTTP请求的处理
* 和内存使用、GC、web请求、数据库资源使用相关的各种指标数据

执行器通过两种方式暴露这些信息：

* web终端（endpoints）
* shell接口

可以打开SSH连接到应用程序，执行命令来检查应用程序的运行。

#### 1.1.3 Spring Boot不能做什么？

* Spring Boot不是一个应用服务器，而是通过嵌入的servlet容器来提供的功能，如tomcat、Jetty等
* Spring Boot没有实现任何企业级Java规范，例如JPA、JMS，它只是利用自动化配置来支持这些功能
* Spring Boot不借助任何形式的代码生成来达到它的目的，而是借助依赖传递来自动化配置beans

### 1.2 开始使用Spring Boot

启动Spring Boot项目最便捷的方法就是安装CLI。

#### 1.2.1 安装Spring Boot CLI

手动安装包下载地址：

[https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.3.1.RELEASE/spring-boot-cli-2.3.1.RELEASE-bin.tar.gz](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.3.1.RELEASE/spring-boot-cli-2.3.1.RELEASE-bin.tar.gz)

安装位置：

```
cd /Users/username/SDevelop/
tar -xzvf spring-boot-cli-2.3.1.RELEASE-bin.tar.gz
ln -s spring-2.3.1.RELEASE spring
```

修改~/.zshrc设置环境变量：

```
export SPRING_HOME=~/SDevelop/spring
export PATH=$PATH:$SPRING_HOME/bin
```

刷新环境变量后查看Spring CLI版本：

```
spring --version
```

Spring CLI v2.3.1.RELEASE

##### 启用命令行Completion

CLI提供了运行、打包、测试等一些命令，每个命令有多个选项。命令行completion可以帮助查看如何使用Spring Boot CLI。

运行命令：

> spring shell

#### 1.2.2 使用Spring Initializr初始化一个Spring Boot项目

Spring Initializr其实是一个web应用，可以生成Spring Boot项目框架。它将为你提供基础项目框架和Maven/Gradle构建规范。

使用Spring Initializr的几种方法：

* 通过一个基于web的接口
* 借助Spring Tool Suite
* 借助IntelliJ IDEA
* 使用Spring Boot CLI

##### 使用Spring Initializr的Web接口

Web接口地址：[http://start.spring.io](http://start.spring.io)

可配置项：

* 构建工具 Maven/Gradle
* Spring Boot版本
* 项目元数据
	* Group
	* Artifact
	* Name
	* Description
	* Package name
	* Packaging -- Jar/War
	* Java version

* Dependencies

因网络问题无法访问start.spring.io，搭建本地Initializr server。

GitHub仓库地址：

> https://github.com/spring-io/initializr.git

```
git clone https://github.com/spring-io/initializr.git
cd initializr/
```
### 1.3 总结


## 2. 开发第一个Spring Boot应用

内容概要：

* 和Spring Boot starters一起工作
* 自动化Spring配置


### 2.1 让Spring Boot开始工作

### 2.2 使用starter依赖

### 2.3 使用自动化配置

### 2.4 总结

## 3. 自定义配置

内容概要：

* 重写自动化配置beans
* 使用外部属性进行配置
* 自定义错误页面

### 3.1 重写Spring Boot自动配置

### 3.2 使用属性（properties）外部化配置

### 3.3 自定义应用错误页面

### 3.4 总结

## 4. 用Spring Boot进行测试

### 4.1 集成测试自动化配置

### 4.2 测试Web应用

### 4.3 测试运行中的应用

### 4.4 总结

## 5. 与Spring Boot CLI搭配使用Groovy

### 5.1 开发一个Spring Boot CLI应用

### 5.2 抓住依赖

### 5.3 使用CLI运行测试

### 5.4 创建一个可部署的工件（artifact）

### 5.5 总结

## 6. 在Spring Boot中采用Grails

### 6.1 使用GORM进行数据持久化
### 6.2 用Groovy Server Pages定义视图
### 6.3 混合使用Spring Boot和Grails
### 6.4 总结
## 7. 窥探Actuator

### 7.1 浏览Actuator的端点
### 7.2 连接到Actuator远程shell
### 7.3 使用JMX监控应用程序
### 7.4 自定义Actuator
### 7.5 加强Actuator端点安全
### 7.6 总结
## 8. 部署Spring Boot应用
### 8.1 权衡部署选项
### 8.2 部署一个应用服务器
### 8.3 推送到云端
### 8.4 总结

## 附录 A：Spring Boot开发工具

## 附录 B：Spring Boot starters

## 附录 C：配置属性

## 附录 D：Spring Boot依赖