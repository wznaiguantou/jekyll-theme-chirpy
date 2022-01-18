---
layout: post
title:  "Kafka概念介绍(一)"
date:   2022-01-19 00:27:58 +0800
categories: Kafka
---

### 1.  什么是Kafka?

先来一段官网的“吹逼”

> Apache Kafka is an open-source distributed event streaming platform used by thousands of companies for high-performance data pipelines, streaming analytics, data integration, and mission-critical applications.

简单翻译下。Kafka就是一个开源的**分布式高性能的事件流**平台。

### 2. 有哪些使用场景

  聊到这个话题，就离不开我们技术人的技术选型这个话题。**事件流平台哪家强？** 凭啥我就选你。

针对这个话题，我们来个5毛钱的唠嗑。 这里不做其他技术选型对比，只突出Kafka。

#### 2.1 特性

-  高吞吐量
-  低延迟
-  高并发
-  消息的持久化和可靠性
-  可扩展性
-  集群容错性

这里小小的乱入一个概念，但是不会展开说:  CAP，Kafka到底是*CP*, 还是*AP*，还是*AC*?

#### 2.2 场景

1. 消息中间件: 解决的就是服务之间的异步通信，从架构上进行解耦
2. 



### 3. 引入一个新的中间件我们该怎么去评估？

我们有哪些维度手段来评估这个组件是否是我们真正需要的？ 这也是一个问题



整理问题：

- 高吞吐量和低延迟是一个意思么？
- 我们有哪些维度手段来评估这个组件是否是我们真正需要的？技术选型的几个维度！
  - 业务驱动？
  - 社区活跃度？
  - 带来的收益 和 副作用？
- 

