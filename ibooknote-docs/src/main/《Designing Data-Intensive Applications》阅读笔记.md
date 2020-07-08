# 《Designing Data-Intensive Applications》阅读笔记

## Preface

Data-Intensive（数据密集型）应用的特征：

* 数据是主要挑战
* 数据质量
* 数据复杂性
* 数据变化的速度

与data-intensive相对应的是compute-intensive（计算密集型）

与data-intensive相关的关键词：

* 新型数据库系统（NoSQL）
* 消息队列（message queues）
* 缓存（caches）
* 搜索引擎（search indexes）
* 批处理框架（batch）
* 流处理

这本书既不是特定工具的帮助手册，也不是干巴巴的纯理论教科书。

## 1、Reliable、Scalable、Maintainable

软件系统最重要的三点：

* 可靠性
* 可扩展性
* 可维护性

### 1.1、Reliability

reliability：continuing to work correctly, even when things go wrong.

fault-tolerant/resilient

fault is not the same as a failure

* fault: 系统的一个组件偏离其预定的规格；缺陷
* failure：整个系统停止为用户提供服务；失败
* fault-tolerance：prevent faults from causing failures.

#### 1.1.1、Hardware Faults

使用多机器冗余，应对单点硬件故障，或进行硬件升级、修复重启。

#### 1.1.2、Software Errors

系统性错误比较难以预见，可能导致多个关联节点出现问题：

* 一个Linux内核的软件bug导致每个服务器实例同时因错误输入而崩溃
* 异常进程耗尽了共享资源--CPU时间、内存、硬盘空间或网络带宽
* 系统依赖的服务响应缓慢或响应错误
* Cascading failures，小错误触发其他组件中的错误，进而出发更多错误

#### 1.1.3、Human Errors

针对大量的internet服务研究发现，人为配置错误导致的运行中断为主要原因，而硬件错误导只占10-25%。

如何保持系统的可靠性：

* 用一种使出错的机会最小化的方法设计系统
* 把最容易出错的地方从他们可能引起failures的地方分离处理
* 在所有级别进行全面测试，从单元测试到整个系统的集成测试和操作测试
* 运行快速从认为错误中恢复，如配置回滚、代码回滚，并提供数据重新计算的工具
* 构建详细、清晰的监控系统，能够监控性能、出错率等
* 实现好的管理实践和训练

cutting corners：偷工减料

### 1.2、Scalability

Scalability: a system's ability to cope with increased load.

* If the system grows in a particular way, what are our options for coping with the growth?
* How can we add computing resources to handle the additional load?

#### 1.2.1、Describing Load

简要的描述系统的当前负载情况：

* web服务器每秒请求次数
* 数据库读、写操作的比例
* 聊天室内同时活跃用户数
* 缓存命中率

#### 1.2.2. Describing Performance

* 当提高负载，而不增加系统资源时，系统性能有何影响？
* 当提高负载，想保持性能不变时，需要增加多少系统资源？

Hadoop这类的批处理系统的性能指标是：吞吐量--每秒处理的记录数据，或者给定数据集上的作业处理时间。对于在线系统来说，性能指标是服务的响应时间。

#### 1.2.3. Approaches for coping with Load

状态数据系统从单节点架构迁移到分布式架构，会引入很多额外的复杂度，为此，以前较明智的做法是先在单节点纵向扩展（scale up），直到有更高要求时在使用分布式架构。

随着分布式系统工具和抽象概念的改进，对一些应用来说，分布式数据系统将是默认的架构，不仅仅是处于扩展性的考虑，还包括易用性和可维护性。

### 1.3. Maintainability

软件系统的三个设计原则：

* Operability 可操作性、可实施性
* Simplicity 简洁
* Evolvability 可进化、可改进

#### 1.3.1. Operability

一个好的实施团队需要完成以下任务：

* 监控系统的健康状况，出现故障时可以快速恢复
* 记录问题原因
* 保持软件和平台的及时更新，包括安全补丁
* 清楚各系统之间的影响，避免一个系统出现问题时，造成其他的损害
* 预见未来可能发生的问题，并能提前解决
* 为发布、配置管理建立良好的实践和工具
* 执行复杂的维护任务，如：平台迁移
* 修改配置时，保证系统的安全性
* 制定标准的生产环境操作流程
* 维护关于系统的组织内部知识库

#### 1.3.2. Simplicity

复杂的标志：

* explosion of the state space
* 模块紧耦合
* 混乱的依赖关系
* 不一致的命名规则和术语
* hacks aimed at solving performance problems
* special-casing to work around issues elsewhere

复杂系统在修改时更容易引入新的bug

removing accidental complexity--abstraction

#### 1.3.3. Evolvability

为适应变化，可采用敏捷开发方法，TDD、重构

## 2. 数据模型和查询语言

relational databases

### 2.1. Relational Model Versus Document Model

#### 2.1.1. The Birth of NoSQL

NoSQL - Not Only SQL

采用NoSQL数据库的原因：

* 大数据集、高写入吞吐量，对扩展性要求更高
* 相比于商业数据库产品，免费、开源更吸引人
* relational model不支持的特殊查询操作
* relational schemas过于严苛的约束，对更动态的、善于表达的数据模型的期待

#### 2.1.2 The Object-Relational Mismatch

对象模型和关系型数据模型之间需要进行互相转换。

### 2.3. Graph-Like Data Models

graph consists of two kinds of objects:

* vertices(nodes, entities)
* edges(relationships, arcs)

examples:

* Social graphs
* The web graph
* Road or rail networks

### 2.4. Summary

介绍各种数据模型：

* 关系型数据模型
* 文档型数据模型
* 图数据模型

每种类型都有其适合的领域，侧重目标不同，所以不同的系统应用不同的数据模型。

## 3. Storage and Retrieval

数据库的两大功能：存储数据、读取数据

storage engines:

* log-structured storage engines
* page-oriented storage engines(B-trees)

### 3.1. Data Structures That Power Your Database

well-chosen indexes speed up read queries, but every index slows down writes.

数据结构：

* Hash Indexes
* SSTables and LSM-Trees
* B-Trees


#### 3.1.1. Hash Indexes

Hash table的限制：

* Hash table必须在内存中使用，不适合在磁盘上，随机读取的I/O开销太大
* 范围查找效率低，必须每个key都找一遍

#### 3.1.3. B-Tree

如何保证B-Trees的可靠性：

* 使用write-ahead log（redo log）解决在更新时系统崩溃之后如何恢复的问题
* 另外一个需要重点关注的是并发访问控制，通常采用锁（latches）机制

B-tree优化：

* 用copy-on-write scheme代替覆盖Pages和WAL，修改的page写在不同的地方，父page指向新地址，这种方法也可用于并发访问控制
* 不存储完整的key，节省空间，保持更大的branching factor
* 顺序的key在磁盘上尽量连续的存储，降低磁盘I/O寻址
* 增加同级指针指向兄弟page，扫描key时可避免跳回父pages
* B-tree的变体借鉴了log-structured概念，降低磁盘寻址

#### 3.1.4. Comparing B-Trees and LSM-Trees

* LSM-trees 写更快速
* B-trees 读更快速

### 3.2 Transaction Processing or Analytics

* OLTP -- Online Transaction Processing, 磁盘寻址是主要瓶颈
* OLAP -- Online Analytic Processing, 磁盘吞吐量是主要平台
* ETL -- Extract Transform Load

## 4. Encoding and Evolution

* 向后兼容 -- 新代码可以读取旧代码写的数据
* 向前兼容 -- 旧代码可以读取新代码写的数据

in-memory to byte sequence: encoding/serializtion/marshalling
byte sequence to in-memory: decoding/deserializtion/unmarshalling

encoding需要考虑的几个问题：

* encoding总是与特定程序语言绑定在一起，需要考虑与其他机构系统集成时的问题
* 为了从数据中恢复对象类型，解码进程需要能够实例化arbitrary classes，可能会有安全问题，被攻击者利用
* 数据版本化经常导致兼容问题
* 效率问题--Java内置的序列化性能很差

这些原因说明使用语言内置的编码/解码功能并不明智。

使用JSON、XML 和Binary Variants可以被多种语言接受，普通文本格式。

相对于JSON、XML、CSV等文本格式，二进制格式编码更节省空间、解析更快速。
二进制编码工具包：

* Thrift -- come from Facebook
* Protocol Buffers -- come from Google
* Avro -- hadoop

SOA - 面向服务架构，构建系统的一种方法
SOAP - 一个特定的技术

## 5. Replication

three popular algorithms for replicating changes between nodes:

* single-leader
* multi-leader
* leaderless

replication需要考虑的问题：

* synchronous or asynchronous
* how to handle failed replicas

semi-synchronous

### 5.3. Multi-Leader Replication

Clients with offline operation:

类似日历应用程序，手机、平台、电脑相当于三个datacenter，当一个处于离线状态，并修改日历时，其数据不能实时同步给其他设备，有工具可以解决这种multi-leader配置问题，例如：CouchDB就是为这种操作模式而设计。

Collaborative editing 协同编辑

冲突解决的方法：

* 给每一个写操作唯一标识 UUID，并根据LWW（last write wins）策略解决冲突，可能导致数据丢失
* 给每一个副本一个唯一的标识，更高的数字具有更高的优先级，也会导致数据丢失
* 将信息合并到一起
* 以额外的数据结构记录冲突，保存所有信息，并通过应用程序来解决冲突，或者有人工解决

冲突解决的逻辑：

* On write 写的时候解决
* On read 读的时候解决，例如CouchDB

## 6. Partitioning

Terminological confusion:

* partition
* shard -- in MongoDB, Elasticsearch, SolrCloud
* region -- in HBase
* tablet -- in Bigtable
* vnode -- in Cassandra, Riak
* vBucket -- in Couchbase

### 6.1. Partitioning of Key-Value Data

* skewed -- 一个分区的数据或查询比其他的多
* hot spot -- A partition with disproportionately high load 不成比例的高负载分区

最简单的避免hot spot的方法：为节点随机分配记录，但这种方法的缺点是读取记录时不知道在哪个节点上，必须并行查询所有节点。

分区方式：

* Partitioning by Key Range
* Partitioning by Hash of Key

### 6.2. Partitioning and Secondary Indexes

### 6.3. Rebalancing Partitions

使用hash值模运算进行分区时的问题：分区后，如果分区数量调整，有些记录要从一个分区转移到另一个分区，解决办法：

* 固定分区数量，初始设计时创建比节点多的分区（如Elasticsearch）
* 动态分区，使用key range-partitioned方式（如HBase，RethinkDB）

尽量用人工参与的方式进行rebalancing，全自动方式容易与自动的故障监测冲突。

### 6.4. Request Routing

服务发现--service discovery

* 允许客户端联系每一个node，如果请求的节点可以处理，直接返回结果，如果不能处理就转发给其他node
* 客户端将请求发给routing tier，有它决定应该请求哪个node
* 客户端清楚每个节点上的分区，知道应该访问哪个一个节点

很多系统使用单独的协作服务，如 ZooKeeper来保存集群元数据。

## 7. Transactions 事务

ACID: Atomicity, Consistency, Isolation, Durability
原子性、一致性、隔离性、持久性

Atomicity, Isolation, Durability 是数据库的属性，Consistency是应用程序的属性。

很多分布式存储抛弃了multi-object transactions，跨分区方式太难实现。


### 7.3 Serializability

#### 7.3.1. Actual Serial Execution

存储过程的缺点：

* 每个数据库提供商都有自己的存储过程语言，没有一个统一的标准
* 在数据库中运行的代码更难管理，包括bug调试、版本兼容问题等
* 同一个数据库可能被多个应用程序同时使用，写的不好的存储过程可能比不好的应用程序引起更多的问题

#### 7.3.3. Serializable Snapshot Isolation(SSI)

悲观锁和乐观锁：

* Pessimistic-- Two-phase locking
* Optimistic-- Serializable snapshot isolation

## 8. The Trouble with Distributed Systems

### 8.1. Faults and Partial Failures

## 9. Consistency and Consensus

一致性和一致意见

### 9.2 Linearizability

* Serializability 事务的隔离属性，每个事务可以读写多个对象（行、文档、记录），一个事务必须在另一个事务开始前结束。
* Linearizability 读写对象的一个最近保障，并不降多个操作组合成一个事务，阻止不了write skew


