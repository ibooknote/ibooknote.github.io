# 《Spring Cloud》官方文档阅读笔记

Spring Cloud为开发者在分布式系统中快速构建一些常用模式提供了工具

* 配置管理
* 服务发现
* circuit breakers
* 智能路由
* 微代理 micro-proxy
* 控制总线

协调分布式系统趋向boiler plate模式，使用Spring Cloud，开发者能够快速建立服务和应用。可以工作在任何分布式环境中，包括开发者的个人电脑、数据中心、管理平台。

boiler plate -- 样板、模版

> standardized pieces of text for use as clauses in contracts or as part of a computer program.

## Chapter 1. 功能

为典型用例提供开箱即用、可扩展的经验：

* 分布式/版本化配置
* 服务注册和发下
* 路由
* 服务间调用
* 负载均衡
* Circuit Breakers 断路器
* 分布式消息

