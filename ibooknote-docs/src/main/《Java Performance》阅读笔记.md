# 《Java Performance》阅读笔记

> Java Performance,In-Depth Advice for Tuning and Programming Java 8, 11, and Beyond(Second Edition)

主要内容：

* 理解各种Java平台和编译器如何影响性能
* 学习Java GC（garbage collection）如何工作
* 性能测试中获得最佳结果的四个原则
* 使用JDK和其他工具学习Java应用如何执行
* 通过tuning和编程实践，最小化gc的影响
* 处理Java API中的性能问题（Tackle performance issues in Java APIs）
* 提升Java驱动数据库应用的性能

## 1 介绍

Java性能的艺术与科学：

* 科学是指其科学的严格性
* 艺术 深入的知识、经验、直觉

本书的宗旨是帮助深入理解Java平台的性能的各个方面。

知识分为两大类：

* JVM的性能 -- JVM的配置影响程序性能的多个方面
* Java平台的功能如何影响性能

platform：一些功能是语言本身的一部分（线程、同步），一些功能是标准Java API的一部分。

JVM性能基于大量的微调参数，而平台性能更多的取决于应用代码的最佳实践。

### 1.1 内容框架

* 第2章，测试Java应用的常用方法
* 第3章，介绍监控Java应用的工具
* 第4章，JIT编译器
* 第5、6章，GC
* 第7章，Java平台的最佳实践--Java堆的内存使用
* 第8章，native内存使用
* 第9章，线程性能
* 第10章，Java服务器技术
* 第11章，数据库访问
* 第12章，常用Java SE API提示

### 1.2 平台和规范

影响Java性能的因素：

* Java自身的版本
* 硬件
* 运行程序的软件平台

#### 1.2.1 Java平台

* HotSpot Java Virtual Machine、JDK、JRE，JRE是JDK的子集
* OpenJDK， AdoptOpenJDK

##### JVM调节标记

两类标记：

* boolean标记 -XX:+FlagName启用标记；-XX:-FlagName停用标记
* 带参数的标记 -XX:FlagName=something

#### 1.2.2 硬件平台

##### 多核硬件

大多数情况下，每个核会包含两个线程（hyper-threading）。当一个线程挂起时，另一个线程执行。

四核8线程，同时只有四个线程在执行状态，其他线程在挂起状态。四核CPU会比单核快5到6倍。

CPU-bound task

##### 软件容器

两种容器：

* 虚拟机 virtual machine
* Docker容器

Java 8以后版本的JVM，当使用Docker容器时，如果Docker本身未设置CPU和内存的大小限制，JVM将使用系统的全部可用资源，如果限制了Docker的CPU、内存，那么JVM将依据这个限制运行。

Java 8以前的版本，即使限制了Docker的资源大小，JVM仍然会计算机器的资源大小，导致启动过多线程和过大的heap。

在Docker容器运行Java的另外一个挑战：Java的很多性能诊断工具在Docker容器下不可用。

### 1.3 完整的性能故事

本书主要讨论如何最优使用JVM和Java平台API，使得程序运行更快，但也有很多外在因素影响性能。

#### 1.3.1 编写好的算法

好的算法对性能很重要

#### 1.3.2 写更少的代码

小的编写良好的程序要比大的编写良好的程序运行的更快：

* 代码越多，编译越慢
* 要分配空间和撤销的对象越多，垃圾收集器的工作就越多
* 要分配空间和保留的对象越多，GC的周期将越长
* 从硬盘加载到JVM的类越多，程序启动就越慢
* 执行的代码越多，能适应机器硬件缓存的就越少
* 执行的代码越多，执行的时间越长

#### 1.3.3 继续，过早的优化

Donald Knuth: 

> We should forget about small efficiencies, say about 97% of the time; premature optimization is the root of all evil.

格言的重点是说应该编写干净、直接、易于阅读和理解的代码。在这种情况下不要过早的优化，除非明确能获得很大的好处。

我们应该避免使用导致性能变差的代码构造。

#### 1.3.4 Look Elsewhere：数据库总是瓶颈

Bugs和性能问题不仅限于JVM

#### 1.3.4 优化常见案例

## 2 性能测试方法

获得性能测试结果的四个原则：

* 测试真实应用
* 理解吞吐量、批量、响应时间
* 理解可变性
* 尽早测试、经常测试

如果没有科学的分析方法，性能测试可能导致不正确或不完整的分析结果。

### 2.1 测试一个真实应用

可用于性能测试的三种代码：

* microbenchmarks 微观基准
* macrobenchmarks 宏观基准
* mesobenchmarks 中观基准

#### 2.1.1 Microbenchmarks

用来衡量小单位的性能，以决定哪个实现更可取：

* 现创建线程 & 线程池
* 执行算数运算算法的时间 & 替代实现

Microbenchmarks是个好主意，但Java的JIT编译和GC使得编写正确的Microbenchmarks比较困难。

#### 2.1.2 Macrobenchmarks

宏观基准需要连同应用其他部分一起考虑。

多JVM共用机器资源时，在CPU繁忙和空闲时，性能测试结果可能不同。

#### 2.1.3 Mesobenchmarks

介于微观基准和全应用之间。

### 2.2 理解吞吐量、批量、响应时间

第二个原则是要理解和选择合适的测试指标。

性能可以通过一下指标衡量：

* 吞吐量（RPS）
* elapsed time（批处理时间）
* 响应时间

三者相互关联。

#### 2.2.1 批处理时间衡量

#### 2.2.2 吞吐量衡量

单位时间内完成的工作：

* TPS - transactions per second
* RPS - requests per second
* OPS - operations per second

#### 2.2.3 响应时间测试

客户端请求和接收到回应的时间间隔

* average response time
* percentile request

平均响应时间受outliers影响较大。

Load Generators 负载发生器：

* fhb（Faban）
* Apache JMeter
* Gatling
* Micro Focus LoadRunner

### 2.3 理解可变性

第三个原则是要理解测试结果随时间变化。

### 2.4 尽早测试，经常测试

性能测试应该作为开发周期的一部分，在提交代码时，自动进行测试。

经常性测试的前提条件：

* 自动化每一件事，性能测试应该脚本化或程序化
* 测量每一种情况
* 在目标系统上运行

经常性测试很重要，但不能凭空产生，必须和正常开发周期保持平衡。
自动化测试系统收集尽可能全面地统计信息，包括机器和程序的，为性能下降分析提供了必要的线索。

### 2.5 Benchmark Examples

jmh

> http://openjdk.java.net/projects/code-tools/jmh/

* Gradle JMH Plugin
* Scala SBT JMH Plugin
* IntelliJ IDEA JMH Plugin
* Jenkins JMH Plugin
* Teamcity JMH Plugin

jmh是编写microbenchmarks的一个框架，位benchmarks需求提供合理的解决办法。

jmh不能替代深入思考编写要测量的代码。

## 3 Java性能工具箱

本章讲解如何获取和理解性能分析数据。

### 3.1 操作系统工具和分析

vmstat、iostat、prstat

#### 3.1.1 CPU使用

CPU使用通常被分为两部分：

* 用户时间 CPU执行应用代码的时间
* 系统时间 CPU执行内核代码的时间

性能目标是驱动CPU使用在尽可能短的时间内，达到尽可能高的使用率。
（反直觉）

5秒或30秒之间CPU的平均使用率。

CPU空闲的原因：

* 应用程序因同步不能执行，被锁
* 应用程序在等待，如访问数据库的回应
* 应用程序没什么可做的

CPU使用率是理解应用程序性能的第一步，但仅此而已：观察代码是否在使用全部的CPU时间，或者指出同步或资源问题。

#### 3.1.2 CPU运行队列

vmstat可以输出运行队列长度

Windows系统使用typeperf

如果队列太长，预示着机器过载，应该减少工作量，或将作业转移到其他机器或优化代码。

优化代码的目标是提高CPU使用率，而不是降低。

#### 3.1.3 磁盘使用

监控磁盘使用的两个重要目标：

* 如果应用有大量磁盘I/O操作，I/O操作很容易成为瓶颈
* 系统产生磁盘交换，当物理内存用尽时

系统名：iostat

#### 3.1.4 网络使用

Unix系统中使用工具netstat，windows系统使用typeperf监控网络使用情况。

命令行工具nicstat

基于网络的应用程序，监控网络确保不会出现瓶颈
应用程序网络写操作的瓶颈原因：写数据效率低、写太多数据

### 3.2 Java监控工具

JDK附带的监控工具：

* jcmd -- 打印指定进程的基本类、线程和JVM信息，可以用于脚本中
* jconsole -- 为JVM的活动提供图形化界面，包括线程使用情况、类使用情况和GC活动
* jmap -- 提供堆转存和JVM内存使用情况的其他信息，可用于脚本中
* jinfo -- 观察JVM的系统属性，有些系统属性可动态设置，可用于脚本中
* jstack -- 转存Java进程栈信息，可用于脚本
* jstat -- 提供关于GC和类加载的信息，可用于脚本
* jvisualvm -- 监控JVM的GUI工具

#### 3.2.1 基础VM信息

* Uptime - 启动时长 VM.uptime
* 系统属性 - System.getProperties()
* JVM版本
* JVM命令行
* JVM调节标记

#### 3.2.2 线程信息

jconsole和jvisualvm显示实时线程数信息。可以用于查看线程是否被锁定。

jstack和jcmd都可以查看栈信息。

#### 3.2.3 类信息

#### Live GC Analysis

#### 3.2.4 堆转存后期处理

heap dump是heap的快照，可用各种工具进行分析，包括jvisualvm。

第三方分析工具：Eclipse Memory Analyzer Tool
（第7章讲详细讲解）

### 3.3 Profiling Tools 分析工具

使用分析工具时需要额外注意，因需要传输大量数据给分析工具，可能影响应用程序。
使用并行GC算法运行分析工具是个好主意。

#### 3.3.1 Sampling Profilers

分析的两种模式：

* sampling mode 采样模式
* instrumented mode 检测模式

safepoint bias：安全点偏差

为分析器提供的Java接口中，分析器只能在线程处于安全点时才能采集到栈追踪信息，线程在以下情况下自动进入安全点：

* 因同步锁被锁定
* 等待I/O被锁定
* 等待监控被锁定
* Parked 寄存
* 执行JNI代码

使用AsyncGetCallTrace接口的分析器可以异步获取JVM栈信息，无需等待线程到达安全点。

基于采样的分析是最常用的方法，对性能影响相对小些。

#### 3.3.2 Instrumented Profilers

检测分析器比采样分析器有更多的影响，但能提供更有益的信息。

通过改变类的二进制码顺序，插入执行计数器。

采样分析定位到类或代码块，检测分析进入代码内部。

#### 3.3.3 Blocking Methods和线程时间表

* 被锁定的线程可能是，也可能不是性能问题，应该检查它们为什么被锁定
* 锁定的线程可以被标识

#### 3.3.3 Native Profilers

原生分析器--分析与原生代码引起的问题。

## 4 Working with the JIT Compiler

### 4.1 JIT编译器

静态编译语言编译后生成二进制码用于执行，编译器会对二进制码进行优化，提高性能，但移植性差，必须针对不同硬件指令集进行编译。

解释性语言是在程序执行时一行一行解释执行，无法在对代码进行优化。性能明显比编译执行慢，但移植性好。

Java试图找到一个折中，Java应用程序不编译成特定CPU的二进制码，而是一种中间状态的低级语言，成为Java字节码。当java程序执行时再编译成特定平台的二进制码（just in time）。

#### 4.1.1 HotSpot编译

通常，程序中只有一小部分代码被频繁执行，应用程序的性能主要依赖于这块代码的执行速度。这块代码被称为hot spots。

JVM执行代码时，不是在一开始就立即编译代码。这有两个原因：

* 如果代码仅被执行一次，立即编译是一种浪费，解释代码要比编译并执行一次快。
* 优化--JVM执行特定方法或循环多次时，JVM可以在编译阶段优化。

对于频繁调用的方法或循环，编译是值得的：一次编译、多次执行比多次解释执行更节省时间。

编译器先执行一遍解释，这样编译器可以指出哪些方法调用频繁。

这种优化必须在代码运行一段时间之后才能做出：这是JIT编译器等待编译代码片段的第二个原因。

##### 寄存器和内存

循环累加和的程序逻辑，和变量如果存储在内存中，每次累加都要从内存中读取，会影响性能，编译器会讲其加载到寄存器中，循环计算完成后，再将计算结果存储到内存。

寄存器使用时编译器的常用优化。

### 4.2 分层编译

老版本JVM需要再编译时指定使用哪个编译器，C1--client、C2--server

在代码执行开始阶段，C1编译器运行较快。
C2在编译代码中作了更好的优化，最终由C2编译器生成的代码比C1编译器生成的代码更快。

Java 8之后的版本开始结合使用C1和C2编译器，程序执行开始阶段使用C1编译器，预热之后，开始使用C2编译器，这个技术被称为--tiered compilation（分层编译）

### 4.3 常用编译器标记（Flag）

#### 4.3.1 调节代码缓冲区

JVM编译代码时，会在代码缓冲区中保持汇编语言指令集。代码缓冲区大小固定，一旦装满，JVM将不能再编译额外的代码。

代码缓冲区过小会有潜在的问题。一些热点方法会编译，但另一些不会：应用程序会运行大量的解释代码。

调节代码缓冲区大小的常用选项是简单的指定默认值的双倍或四倍。

代码缓冲区初始大小是 2496KB，默认最大是240MB。调整缓冲区大小是在后台发生，不会影响性能，所以设置ReservedCodeCacheSize的大小是多数情况下需要的。

Java 11中，代码缓冲区分成三部分：

* 非方法代码
* Profiled code
* Nonprofiled code

nonmethod代码片段按照编译器线程数分配空间，剩下的代码缓冲区由另两个片段平分。
也可以用以下参数分别指定每个部分的空间大小：

* -XX:NonNMethodCodeHeapSize=N: for the nonmethod code
* -XX:ProfiledCodeHapSize=N for the profiled code
* -XX:NonProfiledCodeHapSize=N for the nonprofiled code

代码缓冲区可以使用jconsole实时监控，根据实际情况需要，适时调整缓冲区大小。

#### 4.3.2 检查编译过程

参数：

> -XX:+PrintCompilation

可以看到编译器的工作过程。

编译属性的五个符号：

* % -- OSR编译
* s -- 方法是同步的
* ！-- 方法由一个异常处理
* b -- 编译发生在阻塞模式下
* n -- 编译发生在原生方法的封装

JIT编译是异步过程：当JVM决定某个方法应该编译时，方法被放在一个队列中。JVM继续解释方法，而不是等待编译，然后下一次调用该方法时，JVM将执行编译版本的方法（假设编译完成了）。

OSR -- on-stack replacement

对于长时间运行的循环，JVM注意到循环自身也应该编译，并将这段代码放在编译队列中，但仅仅如此还不够：当循环仍在运行时，JVM必须有能力执行编译版本的循环代码--如果一直等到循环结束，效率会很低（甚至不再有用）。所以当循环代码编译完成时，JVM替换掉stack中的代码，下一个循环迭代将执行更快的编译版本的代码。这就是OSR。

Inspecting Compilation with jstat

如果程序运行时不指定PrintCompilation标记，可以使用jstat观察编译器的工作。

> jstat -compiler 5003

5003表示进程ID。

> jstat -printcompilation 5003 1000

每秒重复一次信息，并打印最后在编译的方法。

#### 4.3.3 分层编译级别

一共有5级编译，C1编译器有3级：

* 0 - 解释代码
* 1 - 简单C1编译代码
* 2 - 限制的C1编译代码
* 3 - 全部的C1编译代码
* 4 - C2编译代码

#### 4.3.4 Deoptimization 反优化

Deoptimization -- 编译器必须撤销之前的优化。

反优化发生在两种情况下：

* code is made not entrant 代码不可进入
* code is made zombie 代码是僵尸

##### Not entrant code

使用接口的时候，根据条件动态判断调用哪个具体实现。
在前一次调用进入的方法做了优化后，再次调用进入了另一个实现方法，那么将取消之前的优化。

第二种不可进入的代码是分层优化。当代码被C2编译器编译时，JVM必须替换掉已经被C1编译器编译的代码。这就需要使旧代码不能进入。

##### Deoptimizing zombie code

被标记为僵尸代码有利于回收代码缓冲区空间，为其他类编译使用。
缺点是一旦被标记为僵尸代码，之后再重新加载，重复使用，JVM就需要重新编译、重新优化代码。

小部分的僵尸代码重新编译并不会对应用造成太大影响。

### 4.4 高级编译器标记（Flag）

介绍一些可以影响编译器的标记。这些标记通常不应该使用。

#### 4.4.1 Compilation Thresholds 编译阀值

什么情况下触发代码编译？主要因素是：代码被执行的频率；当被执行的次数达到编译阀值，编译器认为它有足够信息可以编译代码。

编译基于JVM的两个计数器：

* 方法被调用的次数
* 方法中循环被执行的次数（Branching back）

Branching back -- 循环被完整执行的次数，包括执行到结尾，和执行任意分支，如continue。

当JVM执行一个Java方法时，会检查两个计数器，决定这个方法是否应该被编译。如果是，这个方法将被放入编译队列。这种编译通常称为 standard compilation--标准编译

可以分别设置每层的编译阀值：

> -XX:Tier3InvocationThreshold=N (default 200)  
> -XX:Tier4InvocationThreshold=N (default 5000)  

#### 4.4.2 编译线程

放入编译队列的方法（或循环）专门由一个或多个后台线程处理。

这些队列不是严格的先进先出；谁的执行计数器数值更高，谁有更高的优先级。这种优先级顺序确保最重要的代码最先被编译（这是编译ID不按顺序显示的另一个原因）。

C1和C2编译器有不同的队列，每个队列由不同的线程处理。线程数量由复杂的算法公式计算得出，和CPU核数相关。

编译器线程数可通过以下标记调整：

> -XX:CICompilerCount=N  

JVM用于处理队列的总线程数，在分层编译中，1/3（最少一个）用于处理C1编译器队列，剩下的用于处理C2编译器队列。

什么时候调整线程数？
当在Docker容器中使用老版本的JDK 8时，容器可能因为自动调整出现偏差，这种情况下，应该手动设置标记达到与Docker分配的CPU核数相符的值。

可以通过标记 -XX:+BackgroundCompilation（默认为true）禁用异步编译，禁用后，调用待编译代码的调用方必须等待代码编译完成，而不能直接使用解释器执行。

#### 4.4.3 内联

编译器最重要的一个优化是对方法内联。

现在的JVM会自动对简单方法如get、set方法做内联优化。内联默认时开启的。可以使用 -XX:-Inline标记禁用。禁用内联会使性能降低50%。

决定是否内联一个方法，主要依据：方法的热度（hot）和大小。

一个方法如果小于325字节，并且频繁调用，将会被内联。或者方法小于35字节，则直接内联。

#### 4.4.4 Escape Analysis 转译分析

#### 4.4.5 CPU特定代码


### 4.5 分层编译权衡

javac编译器

* -g 选项包含额外的调试信息，不影响性能
* 使用final关键字不会提高编译代码的速度
* 使用新版本的javac重新编译不会使程序更快

### 4.6 The GraalVM

GraalVM是一个新的虚拟机。不但可运行Java代码，还可运行JS、Python、Ruby、R和传统的JVM字节码。

### 4.7 预编译

* 嵌入式系统没有额外的内存供JIT使用
* 程序在预热之前编译完

标准JDK 11版本开始由预编译功能

#### 4.7.1 Ahead-of-Time Compilation

AOT编译最早出现在JDK 9的Linux版中，11版本在所有平台都支持。但还是改进中。

AOT编译将代码编译成JVM使用的共享库，理论上JIT不需要再被调用。

适合长时间运行的程序，因为启动加载共享库比较耗时。规模较小的程序不会有太多改善。

命令行工具：jaotc

#### 4.7.2 GraalVM原生编译

AOT对相对较大的程序有益，对小程序没有帮助。

GraalVM可以产生完全原生的可执行程序（不需要运行在JVM上）

### 4.8 总结

编译器如何工作的背景。

## 5 介绍垃圾收集器（GC）

调节垃圾收集器对提高Java应用程序性能十分重要。因为Java应用程序的性能严重依赖于垃圾收集技术。

### 5.1 垃圾收集总览

Java程序设计过程中，开发者不需要显示的管理对象生命周期：对象在需要时被创建，当不再使用时，JVM自动释放对象。

调节垃圾收集要比其他语言中的追踪指针bug容易的多。

从基础层面来说，GC由两部分组成：

* 找到对象
* 释放与其相关的内存（不再使用）

JVM定期搜索heap中不再使用的对象。从GC根对象开始，通常包括线程栈和系统类。这些对象总是可以到达，然后GC算法扫描通过根对象能够到达的所有对象。通过GC根对象能够到达的对象是live objects，其他不可到达的对象则是garbage（即使它们保留有其他对象的引用或其他对象引用它们）。

当GC算法找到无用对象时，JVM释放内存，用于其他新增对象。同时还需要对释放的空间进行压缩，防止内存碎片化。

GC：

* 找到无用对象
* 释放可用空间
* 压缩heap

不同的收集器使用不同的方法完成这些操作，特别是压缩：

* 一些算法是延迟压缩，直到确实需要时；
* 一些算法在一个时间点压缩整个sections
* 一些算法通过移动小块内存来压缩heap

不同的算法导致了不同的性能特征。

Java应用中的线程分为两个逻辑分组：

* 一个执行应用逻辑（mutator threads）
* 一个执行GC（GC threads）

GC线程必须确保应用线程不在使用对象时移动对象。

当所有应用线程暂停时，被称为 stop-the-world pauses，这对应用性能有很大影响。
最小化这种暂停是GC调节的重点考虑对象。

#### 5.1.1 分代的垃圾收集器

大部分GC将heap分成几代来工作：

* old generation
* young generation--再被分为eden、survivor spaces

如此分代，是因为很多对象只使用很短的时间。

full GC -- 最简单的GC算法，停止所有应用线程，找到无用对象，释放内存，压缩heap。这种算法通常导致应用线程相对较长时间暂停。

concurrent collectors -- 扫描无用对象时，不停止应用线程，也被称为 low-pause collectors。这种算法暂停时间更短，但需要更多的CPU资源。

选择垃圾合适垃圾收集器需要考虑的重点：

* 单个请求受暂停时长影响时，考虑用concurrent collector
* 平均响应时间更重要的话，noncconcurrent collector更合适
* 如果机器缺少空闲CPU时间，最好使用noncconcurrent collector

#### 5.1.2 垃圾收集算法

GC算法：

* Serial GC
* Throughput（Parallel）GC
* G1 GC
* Concurrent Mark-Sweep（CMS）
* ZGC

##### Serial GC

最简单的收集器。客户类型机器（Windows 32位 JVM）默认的收集器。或单核机器。

serial收集器使用单个线程处理heap。

##### Throughput GC

64位机器，2核或多核CPU，JDK 8 默认使用的收集器。

使用多个线程收集年轻代。使用多个线程处理老年代。

因为使用多个线程，通常被称为parallel collector。

##### G1 GC收集器

concurrent收集策略  

#### 5.1.3 选择垃圾收集算法

选择合适的垃圾收集算法，一部分依赖硬件，一部分依赖应用的类型，也依赖于应用程序的性能目标。

JDK 11中G1 GC是好的选择，JDK 8中依赖应用本身。

G1 GC算法适合当前的大部分应用。

Serial收集算法适合单CPU机器

throughput收集算法适合多CPU机器上运行的批量作业，并且CPU受限的情况。

### 5.2 基础GC调节

基本配置参数对不同的算法通用。

#### 5.2.1 调整Heap大小

对GC的第一个基本调节是应用程序的heap大小。

heap太小，程序将花费太多时间执行GC，而没有足够时间执行应用逻辑。
heap太大，花在GC暂停上的时间就会越多

heap太大的另一个问题是，当系统物理内存不足，开始使用交换内存分区时，可能会被分配到交换空间上，引起性能下降。

第一条原则：不要指定比物理内存大的heap。当多个JVM同时运行时，heap综合不要超过物理内存大小。

控制heap的两个参数：

* -XmsN 初始值
* -XmxN 最大值

heap sizing is one of the JVM's core egonomic tunings.

JVM会根据系统资源情况位heap找到最合适的默认初始值，并根据应用程序对内存的需求情况将heap增长到合适的最大值。

最大值一般控制在物理内存的1/4。

JDK 8以前的版本，JVM使用机器内存总数计算默认值；JDK 8和JDK 11版本中，JVM使用容器限定的内存计算。

如果JVM发现使用初始大小内存，GC操作过多，就会增加内存。

需要确保容器有0.5-1G的内存用于JVM的非heap需要。

最大值不宜设置过大，系统会自动增长到GC算法的性能目标。

#### 5.2.2 调整Generations大小

JVM在确定了heap大小后，会自动决定young和old generation分配多大的空间。

young generation如果过大，young GC暂停时间就会增长，但被收集的次数下降，更少的对象会被移入old generation。

相对的，如果old generation过小，就会很快被填满，导致full GC。

平衡是个关键。不同的GC算法使用不同的方法达到平衡。

命令行标记只设置young generation，剩下的部分全给old generation：

* -XX:NewRatio=N 设置young generation与old的比率
* -XX:NewSize=N 设置young的初始大小
* -XX:MaxNewSize=N 设置young的最大尺寸
* -XmnN 将NewSize和MaxNewSize设置成同样的值

##### Adaptive sizing


#### 5.2.3 Sizing Metaspace 调整元空间大小

当JVM加载类时，它需要保存这些类的元数据。这将开辟一个分离的heap空间，称为metaspace。老版本的JVM中称为permgen。

metaspace中不保存实际的类实例（对象），或者反射对象（方法对象）；

metaspace中的信息只用于编译器和JVM运行时，class metadata。

当调整metaspace大小时，需要full GC，所以这个操作比较代价较大。如果在程序启动时有很多的full GC，可能是因为metaspace正在调整大小，所以增加初始大小是个好办法。


#### 5.2.4 控制并行状态（Parallelism）

除了serial收集器，其他GC算法都适用多线程。线程数由以下参数控制：

> -XX:ParallelGCThreads=N

计算公式：
> ParallelGCThreads = 8 + ((N - 8) * 5 / 8)

* 所有GC算法使用的线程数，基于机器的CPU数量计算
* 多个JVM运行在单个机器上时，GC线程数会太高，必须调低


### 5.3 GC工具

最好的监控方法是查看GC日志，记录了程序执行期间的每个GC操作。

#### 5.3.1 在JDK 8中启动GC Logging

* -verbose:gc
* -XX:+PrintGC
* -XX:+PrintGCDetails 显示更多信息

使用简单日志很难怎断GC时发生了什么。

#### 5.3.2 在JDK 11中启动GC Logging

类似JDK 8中的GC日志格式：

> -Xlog:gc*:file=gc.log:time:filecount=7,filesize=8M

### 5.4 总结

## 6 垃圾收集算法

本章介绍每种收集算法如何工作，以及如何调整能够使其工作的更好。

还介绍了新的、实验性的收集器算法。

### 6.1 理解Throughput Collector

两个操作：

* minor collections 少量收集
* full GCs，标记、释放、压缩目标年代

通过GC日志记录的时间，可以快速确定这种GC收集算法对应用的影响。

#### 6.1.1 Adaptive and Static Heap Size Tuning

重点权衡两点：

* 空间大小
* 暂停时间长短

最佳设置依赖应用的目标。

动态heap调节是很好的第一步。对于很多应用来说已经足够。可以最小化内存使用。

### 6.2 理解G1垃圾收集器

G1 GC被称为concurrent collector，因为释放old generation里的对象和应用线程并行，但不是完全并行，压缩young generation时需要暂停所有应用线程。

四个逻辑操作：

* 一个young 收集
* 一个后台并行的标记循环
* 一个混合收集
* 如果需要，一个full GC

#### 6.2.1 调节G1 GC

调节目标是避免full GC，最小化暂停时间。

调节方法：

* 加大old generation
* 增加后台线程（假设有足够的CPU）
* 更频繁执行G1 GC的后台活动
* 增加混合GC循环中的工作

### 6.3 理解CMS垃圾收集器

### 6.4 高级调节

### 6.5 实验性GC算法

## 7 堆内存最佳实践

两个相互矛盾的目标：

* 节省的创建对象，尽可能更快的释放，使用更少的内存，提高垃圾收集器的效率
* 频繁的重建对象，导致性能降低

### 7.1 堆分析

#### 7.1.1 Heap Histograms 直方图

比heap dumps更少的信息，快速定位到耗费heap空间的的对象。

#### 7.1.2 Heap转存

详细分析GC日志，查找出是哪个Classloader

#### 7.1.3 Out-of-Memory 错误

内存溢出的几种情况：

* 原生内存不足 -- JVM内存需求超过系统的物理内存
* metaspace内存不足 -- metaspace未设置最大值限制，加载太多的类
* Java heap自身内存不足，应用无法创建对象
* JVM花费太多时间执行GC


### 7.2 使用更少的内存

减少内存使用的方法：

* 降低对象大小
* 使用延迟初始化对象
* canonical对象

#### 7.2.1 降低对象大小

* 减少实例变量的数量
* 降低变量的大小

#### 7.2.2 延迟加载


### 7.3 对象生命周期管理

#### 7.3.1 对象重用

JDK和Java程序需要重用的对象：

* Thread pools 线程初始开销比较大
* JDBC pools 数据库连接初始化开销比较大
* Large arrays 数组分配空间时，要求数组内的每一个单独元素必须初始化默认值，大数组会耗费时间
* Native NIO buffers 无论大小，分配的开销都大，最好创建一个大的buffer，分片使用
* Security classes 安全算法对象初始化开销比较大
* String encoder和decoder对象 
* StringBuilder helpers 
* Random number generators Random或者SecureRandom类实例设置种子的开销较大
* Names obtained from DNS lookups DNS域名反查开销大
* ZIP encoders 和decoders 初始化开销不到，但释放的开销大，需要确保释放native memory

对象池和本地线程变量在性能上有差别。

##### Object pools

对象池难于控制合适的大小，开发者还要记得将其放回pool里，而不是直接丢掉。

但对性能方面的主要体现在以下几点：

* GC影响 保持太多对象，会降低GC的效率
* Synchronization 对象池涉及到同步问题，对象频繁的移除、替换，会有竞争，结果导致访问对象池比初始化新对象还慢
* Throttling 缓冲池能够对稀缺资源进行节流控制，如线程、网络连接等

##### Thread-local variables

线程局部变量：

* Life-cycle management 
* Cardinality
* Synchronization

#### 7.3.2 Soft, Weak, 和其他引用

软引用、弱引用 -- indefinite references

专业术语：

* Reference -- strone、weak、soft，一个普通实例变量引用一个对象是强引用
* Indefinite reference -- 不定应用，特殊类型的引用（soft or weak）
* Referent -- 

##### 软引用

软引用的明确含义是什么？？

当对象有机会在将来被重用时，可以使用软引用，但如果最近没有被使用，可以让GC回收对象。（同时考虑当前可用的heap内存大小）。


LRU -- least recently used

不要用太多的软引用，因为它们很容易占满整个heap。当对象数量不是太多时，软引用工作的还好。否则，可以考虑有边界限制的对象池，实现一个LRU缓存。

##### 弱引用

对象只有在弱引用状态下，在每个GC周期才会回收。

与软引用对比：

* 弱引用 -- 如果有人对这个对象感兴趣，那么让我知道它在哪，如果没人需要，就扔掉，需要时再重新创建
* 软引用 -- 在内存足够情况下，尽量保持长时间，直到有人使用或者空间不足被回收

##### Finalizers and final references

实际情况下，使用finalize()方法不是个好主意，尽量避免使用它。JDK 11已经将finalize()方法标记为deprecated。

不推荐使用finalize()不但有功能上的原因，还有性能上的原因，当定义了finalize()方法时，会自动创建Finalizer类对象，引用到finalize()方法，所以目标对象无法被及时释放，只有等到Finalizer对象被回收之后，才能一起被回收。

##### Cleaner对象

JDK 11

#### 7.3.3 Compressed Oops

Oops -- ordinary object pointers

### 7.4 总结

## 8 原生内存最佳实践

heap是Java应用中消耗内存最大者，但JVM也会分配和使用大量的原生内存。

### 8.1 Footprint 

一部分原生内存只有在程序启动时使用，例如加载JAR文件时分配的内存。

#### 8.1.1 Measuring Footprint

Linux上通过ps查看进程内存总量。

RSS -- resident set size

#### 8.1.2 最小化Footprint

限制内存使用的几种途径：

* Heap - 占整个footprint的50%-60%，使用较小的最大heap
* Thread stacks - 线程栈很大，特别是64位JVM，参见Chapter 9
* Code cache - 代码缓冲区使用原生内存保持编译代码（Chapter 4）
* Native library allocations 

footprint的纵大小对性能有很大的影响，特别是在屋里内存受限的情况下。footprint是性能测试需要监控的另一个方面。

#### 8.1.3 原生内存跟踪

选项：

```
-XX:NativeMemoryTracking=off|summary|detail
```

NMT默认是关闭的。如果开启，可以在任何时刻使用jcmd获取原生内存信息。

```
jcmd process_id VM.native_memory summary
```

NMT提供了两个关键信息：

* Total committed size
* Individual committed sizes

##### NMT over time


#### 8.1.4 共享库原生内存

##### Native NIO buffers

写数据给原生NIO buffer，然后发送这些数据给channel（file or socket），不需要在JVM和C库之间拷贝数据。如果使用heap字节buffer，buffer中的内容必须被JVM拷贝。

allocateDirect()方法调用很昂贵，direct byte buffers应该尽量重用。如果原生内存充足，可以给每个线程分配一个direct byte buffer作为线程局部变量，或者使用对象池。

可以设置参数限制直接字节缓存的大小：

```
-XX:MaxDirectMemorySize=N
```

原生内存不能像Java heap那样管理：不能被压缩

### 8.2 JVM Tunings for the Operating System

#### 8.2.1 Large Pages

page - 操作系统用来管理物理内存的一个内存单元。

```
-XX:+UseLargePages
```

### 8.3 总结

## 9 线程和同步性能

### 9.1 线程和硬件

运行在四核八个超线程的机器上的应用，比单核上的性能提高大约5-6倍。

### 9.2 线程池和ThreadPoolExecutors

使用线程池时，调整线程池的大小对获取最佳性能至关重要。过大的线程池会损害性能。

线程池的工作流程：

* 1）Tasks提交给队列
* 2）一定数量的线程从队列中获取任务，并执行它们
* 3）任务的结果返回给客户端或存储到数据库
* 4）完成任务后，线程返回到任务队列获取下一个作业并执行（如果没有更多任务执行，线程开始等待任务）

线程池有个最大值和最小值。因为创建线程成本较高，所以设置了最先线程数，提前创建一些线程；另一方面，线程需要系统资源（包括用于线程栈的原生内存），所以过多空闲线程会消耗太多系统资源，所以最大线程数用于控制一次执行太多任务。

#### 9.2.1 设置最大线程数

给定的硬件，最大线程数设置多少合适？依赖于单个任务被锁的频率。

#### 9.2.2 设置最小线程数

#### 9.2.3 线程池Task大小

任务队列中任务过多，如果线程处理不足可能导致任务执行缓慢。当任务太多时应该新任务应该被拒绝。

#### 9.2.9 调整ThreadPoolExecutor大小

ThreadPoolExecutor依赖于queue的类型来决定什么时候启动一个新线程：

* SynchronousQueue 有新任务时如果无空闲线程，就启动一个新线程，知道达到最大值，这种队列不保留待决任务，如果达到最大值，任务将被拒绝
* Unbounded queues 无限队列，永远不拒绝任务，使用尽可能多的线程。
* Bounded queues 有限队列，

KISS principle: keep it simple, stupid.

推荐用法：尽量不使用Executors类提供的默认无边界线程池，因为其不允许控制应用内存的使用。而应该构建自己的ThreadPoolExecutor，设置相同大小的最大线程和核心线程数，利用ArrayBlockingQueue队列限制内存中等待执行的请求数量。

### 9.3 The ForkJoinPool

ForkJoinPool 使用一个内部无边界list，构造方法中可指定线程数，如果不指定，线程池将使用系统的可用CPU数量，或者容器的可用CPU数量。

ForkJoinPool 类设计采用divide-and-conquer（分而治之）algorithms

将大的数据集合分成两份，分别fork子线程，并把结果合并，如此递归下去，直到达到指定的最小子集。

如果使用ThreadPoolExecutor实现这种递归，父线程需要等待子线程结束才能继续，并且等待过程中线程不能被重用。

而ForkJoinPool在当前任务挂起后，线程可用来执行子任务，执行完子任务再接着执行当前任务，即使递归的子任务很多，也不需要创建太多的线程。

ForkJoinPool真正适合使用的场景：

* 合并部分的算法需要执行一些有趣的工作，不仅仅是将两个数相加
* 算法的叶子计算执行足够的工作，以抵消创建任务的开销

#### 9.3.1 Work Stealing

使用ForkJoinPool这种线程池需要确保任务分割的合理性。

ForkJoinPool的第二个特性是：它实现了work stealing。池中的每个线程有其自己的任务队列（它自己fork的）。线程会优先工作于它自己队列中的任务，一旦队列空了，它们将从其他线程的任务队列中偷取任务。

#### 9.3.2 自动并行化

Arrays类的很多方法使用了并行化：用并行的快速排序算法对数据排序

以可以用于对streams的并行化处理（Chapter 12）。

如果同一个机器上有多个JVM，最好限制每个JVM缓冲池大小：

```
-Djava.util.concurrent.Fork JoinPool.common.parallelism=N
```

### 9.4 线程同步

synchronization：同时只能有一个线程可以访问内存

CAS -- Compare and Swap

虽然CAS指令是无锁的、非阻塞的，但它依然有一些阻塞行为：线程只能以顺序的方式访问受保护的内存。

#### 9.4.1 同步的成本

同步区域的代码从两方面影响性能：

* 应用程序在同步阻塞中花费的时间影响了应用的可扩展性
* 获取同步锁需要CPU周期，进而影响性能

##### 同步和可扩展性

应用程序运行在多线程下的增速定义：

Amdahl’s law（阿姆达尔定律）

Speedup = 1 / ((1 - P) + P/N)

#### 9.4.2 避免同步

如果能避免同步，锁的处罚就不会影响应用的性能。两个常用方法可以用来避免使用同步：

* 在每个线程中使用不同的对象
* 使用CAS-based替代同步

基于CAS的实用程序与传统的同步性能对比：

* 如果访问的资源是非竞争的，基于CAS的保护会略快于传统同步。
* 如果访问的资源有少许或适度的竞争，基于CAS的保护会比传统同步快很多。
* 如果访问的资源有很强的竞争，传统同步在有些情况下是更有效的选择。比如大机器有很多线程。
* 基于CAS的保护不适用于只读的值。

#### 9.4.3 False Sharing

False Sharing -- cache line sharing
虚假共享 -- 当多个变量存储在连续内存区域时，线程加载内存会一次加载相邻附近的其他内容。

虚假共享会严重降低频繁修改volatile变量的代码，或者有同步锁的代码性能。

虚假共享难于发现。

最好将数据存入局部变量，来避免虚假共享。

### 9.5 JVM线程调节

#### 9.5.1 调节线程栈大小

每个线程有一个原生线程栈，OS用于存储线程的调用栈信息。

大部分程序使用256KB的栈就可以运行，个别需要上限1MB。如果设置太小，有很大调用栈的线程可能会跑出 StackOverflowError 错误。

原生内存溢出（无法创建线程）可能预示着三件事：

* 32位JVM机器上，进程存在4GB最大限制
* 系统耗尽了虚拟内存
* 类Unix系统中，用户创建了最大进程数

前两种通过调低栈大小解决，但第三个无效。

修改栈大小：

```
-Xss=N flag (e.g., -Xss=256k)
```

#### 9.5.2 Biased Locking

Biased Locking可能提高缓存命中率。

```
-XX:-UseBiasedLocking
```

默认开启

#### 9.5.3 线程优先级

每个Java线程有一个开发者定义的优先级，可以通过优先级提高某种任务处理线程的性能。但往往不太有效。

操作系统会计算机器上运行的每个线程的当前优先级。当前优先级会考虑Java赋予的优先级，但也会包括很多其他因素。其中最重要的是线程已经持续运行了多久。这样可以确保所有线程都有机会运行。

* Unix系统上，受线程运行时间影响到，Java定义的优先级影响小
* Windows系统上，有更高Java优先级的线程运行的机会更多

不管在哪种情况下，都不能依赖线程优先级，如果任务确实重要，应用逻辑必须自己控制优先级。

### 9.6 监控线程和锁

分析线程和同步的效率，需要重点关注两个指标：

* 线程总数（确保不太高也不太低）
* 线程等待锁和其他资源所耗费的时间

#### 9.6.1 线程可见性

jconsole可以查看JVM中的线程状态。也可以打印单个线程栈。

#### 9.6.2 已锁定线程的可见性

实时线程监视器可以查看应用中运行了那些线程，但并不能提供关于线程所做事情的额外数据。

##### Blocked线程和JFR

JFR - Java Flight Recorder可以查看线程什么时间导致的被锁定

##### Blocked线程和JStack

jstack、jcmd可以提供每个线程的详细状态信息：

* 线程是在运行？
* 是在等待锁？
* 等待I/O？

### 9.7 总结

很少有JVM参数能直接优化线程性能。

好的线程性能是要遵循最佳实践：

* 管理线程数量
* 限制同步的影响

## 10 Java服务器

本章关注的几个常用服务求技术：

* 如何使用不同的线程模型扩展服务器
* 异步响应
* 异步请求
* 处理JSON数据的效率

扩展服务器是关于高效的使用线程：

* 事件驱动
* 非阻塞I/O

传统的Java服务器，如Tomcat、WebSphere等，使用Java NIO APIs达到这个目的。

现在的服务器框架，如Netty、Vert.x，隔离了Java NIO APIs的复杂性，提供抑郁使用的服务器。服务器Spring WebFlux和Helidon都是在这些框架基础上构建的，用于提供可扩展的Java服务器。

这种编程模型给予响应式编程。响应式编程的核心是使用基于事件的范例处理异步数据流。

### 10.1 Java NIO概览

以前版本的Java，所有I/O都是阻塞的。

阻塞模式下，每个客户端连接对应一个服务器线程，每个线程只能处理一个链接。

与每个客户端有关的socket在服务器上注册一个selector（这个selector是Selector类实例，用于处理操作系统的接口，当socket上的数据可用时，提供通知消息）。当客户端发送一个请求，selector从操作系统获取一个事件，然后通知线程池中的一个线程--这个客户端的I/O已经可以读取。线程从客户端读取数据后，处理请求，发送回应，并返回等待下一个请求。

客户端于特定线程解耦。非阻塞I/O可以用更少的线程处理更多的请求，获得极大的效率提升。

### 10.2 服务器容器

扩展服务连接数是服务器性能的第一道障碍，依赖于使用非阻塞I/O作为链接处理。

#### 10.2.1 调节服务器线程池

在非阻塞服务器框架中，一般有一个或多个线程充当selector角色：当I/O可用时，这些线程通知系统调用，被称为selector threads。分离的worker threads线程池在selector通知他们之后，处理实际的请求、回应给客户端。

#### 10.2.2 异步Rest服务器

需要使用异步回应的三个原因：

* 引入更多的并行到业务逻辑中
* 限制活动线程的数量
* 正确的截流服务器

### 10.3 异步外呼（Outbound Calls）

#### 10.3.1 异步HTTP

##### 异步数据库调用

Spring Data R2DBC:

> https://spring.io/projects/spring-data-r2dbc

### 10.4 JSON处理

#### 10.4.1 解析和整理概述

* marshaling - parsing
* unmarshaling

处理JSON数据的三种技术：

* Pull parsers -- pull tokens from parser，最快，适合简单的数据格式
* Document models -- 数据转换成一个文档对象，重要的数据、受保护的内容
* Object representations -- 数据转换成Java对象，最慢，语言级的数据展现形式

#### 10.4.2 JSON对象

创建JSON对象可能导致GC问题，当对象太大时。

#### 10.4.3 JSON解析

Jackson parser通常是最快的parser应该用其代替默认实现。

### 10.5 总结

非阻塞I/O是服务器扩展的基础，它允许服务器可以处理相对更多的链接，而使用相对更少的线程。

## 11 数据库性能最佳实践

### 11.1 简单数据库

### 11.2 JDBC

JPA底层使用的JDBC，理解JDBC性能，有助于使得框架获得更好的性能。

#### 11.2.1 JDBC驱动

##### 工作在哪里执行

* thin driver -- 在Java应用端有很少的工作，大部分都在数据库端执行
* thick driver -- 更多的处理和内存需求是在Java客户端

##### JDBC驱动类型

* Type 1 - ODBC + JDBC 性能最差
* Type 2 - 利用原生库访问数据库，使用C编写
* Type 3 - 纯Java编写，但为特定架构设计，需要使用中间件进行转换
* Type 4 - 纯Java编写，易于使用

#### 11.2.2 JDBC链接池

创建数据库连接需要耗费时间，所以JDBC连接是Java中另一种典型的需要重用的对象。

通常链接池的第一条规则是：为应用中的每个线程创建一个连接。链接池和线程池大小设置相同的值。

需要考虑数据库连接池对GC的影响。

#### 11.2.3 Prepared Statements 和 Statement Pooling

大多数情况下应该使用PreparedStatement，而不是使用Statement：

* PreparedStatement允许数据库重用执行的的SQL信息，这样数据库可以节省一些后续的执行工作
* 并且Prepared Statement有一定的安全性，特别是在指定参数调用时

如果只使用一次的sql，使用regular statement可以节省一些工作。可以让应用运行的更快。

批量操作或重复操作的，最好使用PreparedStatement。

##### Setting up the statement pool

每个连接有一个Prepared statement池，对应一个线程。

##### 管理statement pools

> ConnectionPoolDataSource.setMaxStatements()

Prepared statements会消耗很大一部分heap空间，statement pool必须小心调整。

#### 11.2.4 Transactions 事务

数据库事务有两个性能损耗：

* 花费时间用于数据库建立和提交事务 -- 确保修改完全存储到磁盘上，数据库事务日志的一致性
* 在数据库事务过程中，事务通常需要获取特定数据集的锁

##### JDBC事务控制

在JDBC和JPA应用程序中都有事务，但JPA管理事务的方式不同。对于JDBC来说，事务都是在Connection对象使用过程中开始和结束的。

通常情况下，使用JDBC时，默认设置自动提交，每条statement都有其自己的事务。

如果自动提交被关闭，事务将在第一次调用connection对象时隐式的开始。并一致持续到commit()方法被调用（或者rollback()方法）。新事务将在下一次数据库调用时开始。

事务提交开销较大，所以一个目标是在同一个事务中尽可能执行较多的工作。但事务需要占用锁，所以另一个目标是尽可能的短时间。二者之间要达到一个平衡。

##### 事务隔离和锁

基础事务隔离形式（从上到下开销逐渐减小）：

* TRANSACTION_SERIALIZABLE 事务中所有被访问数据都被锁定，包括通过主键和WHERE条件
* TRANSACTION_REPEATABLE_READ 访问的数据在事务过程中都被锁定。但其他事物可以在任意时刻插入新纪录。这可能导致幻读（phantom reads）：一个事务中的查询可能在两次请求中得到的结果不同。
* TRANSACTION_READ_COMMITTED 只锁定在事务过程中写入的行。这导致不可重复读（nonrepeatable reads）：在事务中一个点读取的数据可能和在另一个点读取的不同。
* TRANSACTION_READ_UNCOMMITTED 开销最小的事务模式。不需要锁，一个事务可以读取另一个事务写入但还没有提交的数据。这被称为脏读（dirty read）

各种数据库的默认事务隔离模式：

* MySQL -- TRANSACTION_REPEATABLE_READ
* Oracle、Db2 -- TRANSACTION_READ_COMMITTED

Oracle不支持TRANS ACTION_READ_UNCOMMITTED 或者 TRANSACTION_REPEATABLE_READ


#### 11.2.5 结果集处理

如果查询结果集一次性都返回给应用程序，应用需要在heap中保持一个非常大的块用于存储数据，这可能导致GC问题。相反的，如果每次调用next()只返回一条，在应用和数据库之间将产生大量的来回通信。

通常的做法是设置PreparedStatement对象中的setFetchSize()，让JDBC驱动知道一次传输多少条记录。

### 11.3 JPA

#### 11.3.1 优化JPA写

JDBC中两个重要的性能优化技术：

* 重用prepared statements
* 批量执行updates

Writing Fewer Fields：只写那些有变化的字段。

JPA的优化：

* JPA的实现应该能够提供对字段值是否发生变化的跟中，这样可以只更新发生变化的字段。
* Statement缓存
* 批量执行JPA更新

#### 11.3.2 优化JPA读

什么时候以及如何优化JPA读数据，比看上去更复杂，因为JPA会缓存数据，以便将来可能会使用。

JPA会在三种情况下从数据库读取数据：

* 调用EntityManager的find()方法
* JPA查询被执行
* 代码通过已存在的实体指向一个新的实体

##### 读取更少的数据

对于BLOB- 或者CLOB- 类型的对象，可以使用注解设置其延迟加载（只在需要的时候），这样使得SQL更快，节省大量内存，减轻GC压力。

延迟加载注解只是对JPA实现的一个提示（建议）

eager fetching -- 当一个实体被获取到，其他关联实体也应该被返回。

##### 在查询中使用联合


#### 11.3.3 JPA缓存

为了提高性能，Java提供一个中间层用于缓存从数据库资源获取的数据。

JPA中有两种类型的缓存：

* L1 - Level 1，实体管理器实例（entity manager instance）自己的缓存
* L2 Level 2，全局缓存，在所有实体管理器之间共享

##### 一级缓存

实体管理器自己的缓存，在一个事务过程中获取的数据，被局部缓存起来，也可能包括事务中写的数据。这些数据只有在事务提交时，才发送给数据库。

一个程序中可能有很多歌实体管理器实例，每个执行一个不同的事务，每个有其自己的局部缓存。

##### 二级缓存

当实体管理器提交一个事务时，其中的所有局部缓存数据可以被合并入全局缓存。这个全局缓存在所有的实体管理器间共享。全局缓存也被称为二级缓存。

一级缓存中没有什么可调整的，它在所有JPA实现中都是启用的。但二级缓存有些不同：大部分JPA实现都提供了二级缓存，但并非所有的都是默认开启。

一旦启用二级缓存，并对其进行合理的优化后，可对性能有客观的影响。

JPA缓存只对通过主键访问的实体起作用。

除非JPA实现支持查询缓存，否则使用联合查询会影响性能，因为它绕过了L2缓存。

注：缓存部分、一级缓存、二级缓存需要重点服务、理解。

### 11.4 Spring Data

Spring Data是关系型数据库和NoSQL数据库访问模块的集合，该框架包括以下几个模块：

* Spring Data JDBC -- JPA的简单替代方案，类似JPA，但不提供缓存、延迟加载和dirty实体跟踪，必须在代码中自己控制
* Spring Data JPA -- 在标准的JPA基础上进行了封装，减少了开发者需要编写的代码量
* Spring Data for NoSQL -- 集成了各种NoSQL连接器，MongoDB, Cassandra, Couchbase, Redis
* Srping Data R2DBC -- 允许异步JDBC访问，Postgres、H2、SQL Server，没有缓存、延迟加载等特性


### 11.5 总结

调整访问数据库的JDBC和JPA是应用中间层性能优化的重要方法，最佳实践如下：

* 尽可能使用批量读写
* 优化应用的SQL，使其可以使用L2 缓存
* 最小化锁应用，对于那些不太可能竞争的数据使用乐观锁，对于会发生竞争的使用悲观锁
* 确保使用prepared statement缓冲池
* 确保使用适当大小的链接池
* 设置合适的事务范围：在不影响应用程序可扩展性的前提下，尽可能大

## 12 Java SE API Tips

介绍Java SE API中影响性能的部分：

* 字符串处理
* 适当使用buffer I/O
* 类加载和提供应用程序启动性能的方法
* 适当使用集合
* JDK 8的特性功能 lambdas、streams

### 12.1 Strings

String是java中最常用的对象。

#### 12.1.1 字符串压缩

Java 8中，所有字符串都编码为16位字符的数组，不管字符串本身使用什么编码。这很浪费空间，大部分西方字符可以编码为8位字符数组。

Java 11中，字符串被编码为8位字符数组，除非明确需要16位字符。这些字符串被称为compact strings。（与Java 6中的compressed strings实现上有区别）。

因此Java 11中的字符串通常只有Java 8中一般空间大小。

#### 12.1.2 Duplicate Strings and String Interning

如果有大量重复字符串，需要做heap分析，可以按如下步骤使用Eclipse Memory Analyzer:

* 1 加载heap dump
* 2 在查询浏览器中选择Java Basics -> Group By Value
* 3 设置objects参数位java.lang.String
* 4 点击完成按钮

重复字符串可以通过以下三种方法移除：

* 通过G1 GC自动执行去重
* 使用String类的intern()方法创建string的权威版本
* 使用自定义方法创建string的权威版本

##### 字符串去重

最简单的方法是设置JVM标记：

```
-XX:+UseStringDeduplication
```

默认未开启：

* 因为它在G1 GC的young时期和混合阶段，需要额外处理
* 另外需要额外的线程
* 第三，如果去重字符串很少，应用程序使用的内存将会过高，而不是低，这些内存用于跟踪记录字符串，以查找重复。

##### 字符串interning

Interned字符串存储在原生内存的特定哈希表中（字符串自己仍然在heap中）

可以使用以下参数观察string table的执行情况：

```
-XX:+PrintStringTableStatistics
```

#### 12.1.3 字符串拼接

Bottom line：在单行语句中使用加号直接拼接字符串不用担心性能问题，JVM 8、11会对其进行优化。但如果在循环中使用拼接，就不适合使用加号拼接，而应该总是使用StringBuilder对象，以获得更好的性能（StringBuilder对象在循环外声明）。

单行拼接字符串时，如果含有double类型，最好在JDK 11中重新编译，可以获得更好的性能，因为JDK 8版本下如果拼接double到字符串中，JVM不会对double类型的做特殊优化。

### 12.2 Buffered I/O

InputStream.read() 和 OutputStream.write()两个方法操作单个字符，都比较慢。FileInputStream.read()方法更慢：每次方法执行，都从内核获取一个字节数据。

基于文件的I/O使用二进制数据：

* BufferedInputStream
* BufferedOutputStream

基于文件的I/O使用字符（字符串）数据：

* BufferedReader
* BufferedWriter

在bytes和characters之间进行转换时，操作的数据块越大，性能越好，逐个字节或字符操作，性能很差。

与buffered I/O相关的问题，通常都是由默认的简单input、output流实现类引起的。

当对数据进行压缩或编码时，文件和socket网络I/O需要使用适当的缓冲。

### 12.3 Classloading

#### 12.3.1 Class Data Sharing

CDS - Class data sharing 类的元数据在JVM虚拟机之间共享的一个机制。

运行多个JVM时，CDS可以节省内存空间。

对于单个JVM来说，CDS也可以加快它的启动。

### 12.4 随机数

Java自带的三种标准随机数生成器类：

* java.util.Random，带锁，多线程时会出现竞争
* java.util.concurrent.ThreadLocalRandom，每个线程有其自己的随机数生成器
* java.security.SecureRandom，使用的算法不同，上面两个使用的是伪随机数算法

伪随机数算法必须设定seed，容易被黑客算出下一个将产生的数。

SecureRandom类使用系统接口获取一个随机数数据的种子。在特定的操作系统下基于真正的随机事件生成的随机数。也被称为 entropy-based randomness，相对更安全。

/dev/random 用于获取种子
/dev/urandom 用于获取随机数

SecureRandom.generatedSeed() 花费的时间不定，取决于系统有多少未使用的entropy，如果entropy不可用，调用将被挂起，知道有entropy。所以其性能也变得随机。

generatedSeed()方法常用于以下目的：

* 一些算法用其提前获取种子，为将来调用nextRandom()时使用。一次性或周期性调用
* 创建长期存在的key时使用。

对于云环境下的主机操作系统随机数设备是在多个虚拟主机或Docker容器下共享的。这种情况下，为了使用安全的种子，程序启动时可能会很慢。

程序代码应该运行性能测试，以确定在生产环境下是否能获取到足够的随机数种子。

另一种方式是使用/dev/urandom来获取种子，有两种设置方法：

* -Djava.security.egd=file:/dev/urandom
* $JAVA_HOME/jre/lib/security/java.security (securerandom.source=file:/dev/urandom)

更好的解决办法是配置操作系统来提供更多的entropy，可以启动rngd daemon。但需要确保使用/dev/hwrng，而不是/dev/urandom。这种方式可以解决机器上所有程序的entropy问题，不仅仅是Java程序。

Java的默认随机数类初始化时开销较大，但初始化后可重复利用。

### 12.5 Java原生接口

尽可能避免从Java中调用C，跨JNI边界开销很大。

有趣的是，反过来C代码调用Java却不会有很大的性能影响。

* JNI不是解决性能问题的一个方法。Java代码通常都比调用原生代码更快。
* 使用JNI时，尽量限制从Java调用C的次数，开销很大。
* 使用数组或字符串的JNI代码必须将那些对象固定。

### 12.6 异常

因为异常处理的开销要比正常流程的大，所以不应该将异常处理用于正常的业务逻辑：

* 代码应该只有在不可预知的事情发生时才抛出异常。

异常处理有两点可能影响性能的地方：

* 建立try-catch块开销是否昂贵？现代JVM已经使得处理异常的效率很高了
* 异常需要获得栈追踪信息，这个操作开销较大，特别当栈很深时


### 12.7 日志

日志记录的越少越好。

HTTP访问日志最好只记录必要的字段：

* 使用IP地址，而不是主机名
* 使用时间戳而不是字符串格式时间
* ...

应用程序日志的三哥基本原则：

* 在日志记录数据和日志级别间保持平衡，JDK有7个标准级别
* 使用细粒度的日志
* 日志代码很容易引入意外副作用，即使没有开启--为了判断是否开启日志，执行方法调用

### 12.8 Java集合API

Java有58个集合类。使用合适的集合类，对应用程序的性能很重要。

* 第一条规则是使用一个集合类时，要适合应用的算法需求。（不只是Java）
	* LinkedList不适合搜索；
	* 随机访问数据，应该使用HashMap；
	* 如果数据想保持顺序，应该使用TreeMap，而不是在应用程序里排序；
	* 如果数据经常通过索引访问，并且不会经常往中间插入，使用ArrayList；

使用Java集合时需要考虑的一些特性：

* 同步 & 非同步
* 集合大小
* 集合与内存效率

对于只有一两个元素的集合，如果使用的地方很多，会很浪费内存，这种情况可以考虑初始化时指定大小，或者考虑是否真有必要在这种情况下使用集合。

集合排序时，如果数组很小（少于47个），最快的方法是使用插入排序，其他的是使用快速排序。

### 12.9 Lambdas和匿名类

lambda代码创建一个静态方法，通过特定的帮助类调用，而匿名类是一个实际的Java类，有独立的类文件，从classloader加载，如果有大量匿名类的程序启动时会影响性能。

选择使用lambda还是匿名类，主要看编码的易用性，他们本身在运行时性能没什么差别。

### 12.10 Stream 和过滤器性能

Java 8中经常与lambdas配合使用的关键功能是Stream，它的一个重要性能特征是可以自动并行化代码。

#### 12.10.1 Lazy Traversal

lazy data structures（lazy数据结构）

* Stream过滤器有显著的性能优势，它运行在数据迭代中途结束。
* 即使便利完整的数据，单个过滤器的性能也会比显示的循环更优。
* 多个过滤器有一定的开销，需要确保编写合理的过滤器。

### 12.11 对象序列化

对象序列化是将对象的二进制状态写出，以后再重新创建对象。

#### 12.11.1 Transient Fields

序列化尽量少的数据，可以将不需要的字段标记为 transient。

#### 12.11.2 重写默认序列化

需要注意的是，优化序列化的性能，需要对对象引用做特殊处理。优化好了确实可以很多程度提升性能，优化不好可能会引入bug。

#### 12.11.3 压缩序列化数据

在序列化使用压缩时需要考虑具体场景，如果是直接写入byte stream，采用压缩方式可能会花更多时间。可以考虑采用延迟解压。

如果目的是节省内存空间而不是时间，可以考虑使用压缩和解压

#### 12.11.4 跟踪重复对象

### 12.12 总结
