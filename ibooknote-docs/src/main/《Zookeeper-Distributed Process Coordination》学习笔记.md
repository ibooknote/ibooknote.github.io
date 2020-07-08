# 《Zookeeper-Distributed Process Coordination》学习笔记

## 1、前言

Part I 介绍ZooKeeper诞生的原因，分布式系统的背景知识。

Part II 介绍ZooKeeper编程接口，Java API、C API等，系统管理员可以可以略过。

Part III 讲解ZooKeeper的系统管理员需要的知识，程序员也应该了解，特别关于internals。

## 2、问题

ZooKeeper要解决的问题？

ZooKeeper如何解决分布式系统问题？

Zookeeper本身的管理问题？

## 3、学习内容

### 3.1、Chapter 1 介绍

大数据和云计算的时代，相互独立的应用程序之间协同工作比较困难。

ZooKeeper设计用来让开发者只关注应用逻辑，而无需考虑协同。Zookeeper暴露简单API，允许开发者实现协同任务，例如：选举Master server，管理集群之间的成员、管理元数据。

Zookeeper是一个应用程序库，提供Java和C API，和一个用Java实现的运行在一个团队服务器上的服务组件。

#### 3.1.1、ZooKeeper的任务

enables coordination tasks for distributed systems

ZooKeeper应用场景：

* Apache HBase--选举集群master、跟踪可用的server、保持集群元数据
* Apache Kafka--检测宕机、发现topic，管理topic的生产者和消费者
* Apache Solr--在SolrCloud中ZooKeeper用于存储集群元数据，协调元数据的更新
* Yahoo! Fetching Service--master选举、crash监测、元数据存储
* Facebook Messages--分片和故障转移、服务发现

应用程序作为客户端，连接ZooKeeper服务器，并通过API执行操作。ZooKeeper API提供以下功能：

* 强统一性、顺序性、持续性保证
* 实现典型的同步原语的能力
* 处理分布式系统中的并行问题

ZooKeeper专注于协同任务。

ZooKeeper做不到的事情：

* 不能用于存储大量数据

ZooKeeper只提供实现协同任务的工具，由开发者决定实现什么样的协同任务。

##### 3.1.1.4、利用ZooKeeper构建分布式系统

分布式系统中的通信方式有两种：

* 通过网络直接交换信息
* 从共享存储中读写信息

ZooKeeper使用的是共享存储模型。但共享存储本身与应用处理过程之间也需要网络通信，所以在设计分布式系统时要小心以下问题：

* 消息延迟--由于网络拥堵导致的延迟，可能使得消息的前后顺序发生变化。
* 处理速度--操作系统的调度和过载，可能导致消息处理延迟（接收、传输、处理）。
* Clock drift--时钟偏移，通信双方之间的时钟不一致。

这种延迟的问题在实践中比较难以定位具体的原因--asynchronous

ZooKeeper不能使分布式系统的问题消失，但能使问题更容易跟踪。

#### 3.1.2、举例：Master-Worker应用

要实现master-worker系统，我们必须解决三个关键问题：

* Master crashes--Master发生错误的话，系统无法分配新任务、不能从失败的worker中恢复任务
* Worker crashes--Worker发生错误，分配的任务无法完成
* Communication failures--Master和Worker无法交换信息，worker无法获取分配给它的任务

为解决这些问题，系统必须在master出错时能够重新选举master，并能确认哪些worker可以工作，决定worker状态什么时候需要更新。

##### 3.1.2.1、Master失败

为了应对Master失败，我们需要有一个备用master，当主master失败时，备用master充当主master角色。新的主master必须能够恢复原主master故障时的系统状态。为了恢复master状体，我们不能寄希望于拉起失败的master，因为它已经宕掉了，为了达到这个目的，我们可以使用ZooKeeper。

从状态中恢复不是唯一问题。当备用master错误推测主master出现故障时（主master过于繁忙，消息发生延迟），备用master会自动以主master角色执行，编程第二个主master。更糟糕的是，如果workers不能与主master正常通信，他们可能会转而跟随第二主master。这种情况导致的问题通常称为“split-brain”（精神分裂症）

##### 3.1.2.2、Worker失败

客户端提交任务给master，master将任务分配给可用的worker。worker接收到任务之后，将执行结果上报给master，master再通知客户端执行结果。

如果worker crashes，分配给它的和未完成的任务必须被重新分配。这就要求master能够检测worker是否crash，并能决定其他worker是否能代替执行任务。即使worker执行完任务，但没有上报结果而crash了，系统需要清除状态。

##### 3.1.2.3、通信失败

如果因为网络堵塞，worker无法与master联系，重新飞配的任务可能导致两个worker执行同一个任务。如果重复执行任务是可接受的，那么重新分配前无需验证前一个worker是否已经执行完任务。如果不可接受重复，那么应用程序必须能够处理这种重复执行。

为了解决通信失败的问题，ZooKeeper实现的机制是：

* 确保客户端知道ZooKeeper中的一些数据状态是短暂的
* ZooKeeper ensemble要求客户端必须定期通知是否还活着。如果客户端在规定的时间内通知失败了，属于它的所有临时状态都将被删除。

#### 3.1.3、为什么分布式系统中的协同很难？

* 网络堵塞无法完全避免
* 时钟偏移
* Byzantine Faults

不存在完美的解决方案。

#### 3.1.4、ZooKeeper是成功的，但需要警惕

ZooKeeper不是要解决分布式应用开发面对的所有问题。但它为开发者提供了一个优秀的框架来处理这些问题。

这本书的目的是要使读者理解使用ZooKeeper时应该做哪些事情，以及为什么要那样做。


### 3.2、Chapter 2 Grips with ZooKeeper

#### 3.2.1、ZooKeeper基础

primitive -- 原语

分布式锁：

* create
* acquire
* release

ZooKeeper暴露了一组类文件系统API--recipes

需要重点注意的一点：ZooKeeper不允许部分的写或读znode数据。

##### 3.2.1.1、不同类型的Znode

* 持久的znode -- 只能通过调用delete删除
* 临时的znode -- 如果创建它的客户端崩溃或与ZooKeeper失去联系，就会删除；不能有子节点

four modes：

* persistent
* ephemeral
* persistent_sequential
* and ephemeral_sequential


#### 3.2.2、ZooKeeper架构

ZooKeeper服务器有两种运行模式：

* standalone - a single server, ZooKeeper state is not replicated.
* quorum - a group of ZooKeeper servers(ZooKeeper ensemble)同步状态，一起处理客户端请求.

##### 3.2.2.1、ZooKeeper Quorums

quorum是ZooKeeper能够正常工作的最小服务器数量。

#### 3.2.3、Getting Started with ZooKeeper

#### 3.2.3.2、Session状态和生命周期

session的集中主要状态：

* CONNECTING
* CONNECTED
* CLOSED
* NOT_CONNECTED

当客户端有多个选择连接目标时，一定选择比上一次更新位置新的服务器进行连接。

#### 3.2.3.3、ZooKeeper with Quorums

可以在单个机器上运行多个ZooKeeper服务器，只是需要较为复杂的配置。
每个实例指定不同的端口，启动时指定配置文件。

#### 3.2.3.4、Takeaway Messages

Zookeeper中的两个重要概念：

* quorums
* session

### 3.3、Chapter 3 ZooKeeper API

介绍如何使用ZooKeeper API编程，创建session、实现一个watcher。

Java best practices dictate:

> other methods of an object should not be called until the object's constructor has finished.

#### 3.3.2、创建ZooKeeper Session

##### 3.3.2.2、运行Watcher

当看到Disconnected事件时，不要重新创建新的ZooKeeper handle 来重新连接服务。ZooKeeper客户端库会帮助重新连接服务。当网络中断或服务器发生错误，ZooKeeper能够处理这些失败。



