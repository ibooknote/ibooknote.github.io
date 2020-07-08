# Apache JMeter学习

测试功能和性能。主要为Web 应用提供测试服务。

## 1、能干什么

测试静态、动态资源，Web动态应用。

可以用于模拟高负载服务器、服务器集群、网络或对象测试，用以分析不同类型环境下的整体性能。

功能包括：

* 加载、测试不同类型的应用、服务器、协议：
  * Web - HTTP，HTTPS
  * SOAP/REST Webservices
  * FTP
  * Database via JDBC
  * LDAP
  * 消息中间件（JMS）
  * Mail
  * 原生命令、shell脚本
  * TCP
  * Java对象

* 带有测试IDE环境，录制测试计划（浏览器、原生应用）、构建、调试
* 命令行模式（CLI mode）
* 可以生成完整的HTML形式 测试报告
* 能从response中解析数据
* 可移植性--100%纯Java
* 多线程框架--可以模拟多线程，或分成多个线程组
* 可以缓存或离线分析、回放测试结果
* 高可扩展性
  * 插件化，扩展测试能力
  * 脚本编写简单，类似Groovy
  * 多种计时器插件可选
  * 个性化的数据分析、可视化插件
  * 提供动态数据输入、数据操作功能
  * 配合Maven、Gradle、Jenkins实现持续集成

## 16、最佳实践

### 16.1、总是使用最新版JMeter

新版JMeter总是会有性能提升，尽量避免使用比最新版老3个版本的。

### 16.2、使用正确的线程数

硬件处理能力会影响有效的线程数。线程数还依赖于服务器的性能。如果线程数与硬件不匹配，可能导致协同遗漏（Coordinated Omission）。如果需要大量的加载测试，考虑在多台服务器上使用分布式模式，运行多个CLI JMeter实例。

当使用分布式模式时，测试结果文件会在控制器节点（master）上合并。

JMeter允许使用延迟创建线程的选项。

### 16.3、Cookie管理

参考Building a Web Test：

> https://jmeter.apache.org/usermanual/build-web-test-plan.html#adding_cookie_support

### 16.4、权限管理

参考 Building an Advanced Web Test：

> https://jmeter.apache.org/usermanual/build-adv-web-test-plan.html#header_manager

### 16.5、使用HTTP(S)测试脚本记录器

### 16.6、使用变量

### 16.9、BeanShell脚本

## 25、JMeter分布式测试

使用多个服务器执行压力测试，开始之前需要检查的事情：

* 系统防火墙是否已关闭，或者目标端口是否已打开（有外网IP的服务器不建议关闭防火墙，可能在很短的时间内被攻击）
* 所有客户端在相同的子网内
* 服务器也应该在相同的子网内
* 确保JMeter可以访问服务器
* 确保在各个系统上使用相同版本的JMeter，混合版本可能会出错
* 确保为RMI安装了SSL，或禁用该功能

在所有客户端系统上安装JMeter。JMeter的工作方式是通过一个Master控制多个Slave系统。

在实际测试时应该使用CLI mode，不要使用GUI mode。

### 25.1 术语

* Master - 运行JMeter GUI，控制测试
* Slave - 运行 jemter-server，从GUI接收命令，并发送给目标系统
* Target - 要进行压力测试的目标服务器

### 25.2、Step-by-Step

* 在Slave系统上启动jmeter-server
* 在Master系统上编辑jmeter.properties，将Slave服务器的IP加到remote_hosts上
* 启动JMeter GUI
* 打开测试计划

### 25.3、启动测试

查看jmeter-server.log确认slave是否工作正常。

### 25.4、启动单个客户端

在 Run-Remote Start可以启动单个slave系统进行测试。

### 25.5、启动所有客户端

在 Run-Remote Start All 可以启动所有slave系统进行测试。

### 25.6、限制

分布式测试的一些限制：

* RMI（远程方法调用）跨子网通信必须使用代理
* 2.9版本以后JMeter会把所有测试结果数据发到控制台，这样可以减少对network IO的影响。所以需要监控网络状况，避免被其他情况干扰。
* 单个 JMeter客户端运行在一个2-3GHZ的CPU上，可以处理1000-2000个线程。

### 25.7、相关资源

Wiki page on remote testing

> https://cwiki.apache.org/confluence/display/JMETER/JMeterFAQ#JMeterFAQ-Howtodoremotetestingthe'properway'?

Remote Testing in the user manual

> https://jmeter.apache.org/usermanual/remote-test.html

### 25.8、Tips

有些情况下，防火墙可能会阻止RMI通信。

#### 25.8.1、杀毒软件和防火墙

测试期间，应该暂停杀毒软件，否则可能影响测试，导致错误的结果。

测试期间需要关闭防火墙，或者打开指定的端口。

## 26、JMeter HTTP(S)测试脚本录制

### 26.1、JMeter配置

JMeter 2.10之后，录制功能需要使用JRE/JDK中的 keytool工具箱。

### 26.2、基础指令

* 启动jmeter
* 选择模版菜单
* 选择录制模版
* 创建测试计划
* 设置服务器名称或IP、路径
* 在浏览器中安装crt证书

### 26.3、配置浏览器，使用JMeter Proxy

在浏览器高级配置中设置HTTP Proxy代理，localhost 88888，选择“Use this proxy server for all protocles“。

### 26.4、录制导航

* 在浏览器地址中输入网站地址，按回车键
* 在网页中点击一些连接
* 关闭浏览器，打开JMeter窗口

展开Thread Group后，可以看到几个例子，这个时候可以保存测试计划。

Thread Group 可以配置的参数：

* 线程数（用户数）
* Ramp-Up Period（秒）
* 循环次数

### 26.5、验证脚本

测试之前，验证脚本，Thread Group-Validate

### 26.6、为脚本设置变量、关联数据

有些情况下，需要：

* 设置输入变量（用户名、密码、关键词）
* 两次访问间的关联数据（session变量）

为了设置变量可以使用：

* CSV 数据配置，从CSV文件中读取输入数据。
* JMeter函数（__counter, __time,...)

关联数据可以从一个请求获取数据，然后注入另一个请求中。

### 26.7、启动测试

启动测试的两种方法：

* GUI启动（不推荐在大负载情况下使用），调试阶段。
* 命令行启动

命令行启动命令：

> jmeter -n -t [jmx file] -l [results file] -e -o [Path to output folder]

命令行方式测试后，会生成HTML格式测试报告。
