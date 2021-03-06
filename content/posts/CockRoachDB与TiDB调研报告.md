---
title: CockRoachDB与TiDB调研报告
date: 2018-08-28T09:21:30+08:00
draft: false
toc: true
comments: true
aliases:
  - /detail/166
  - /detail/166/
tags:
  - sql
---

## 基本概念

### “内部一致性”和“外部一致性”

* 内部一致性：
  * ACID中的一致性，核心是约束（唯一、完整性约束）
  * 单机数据库
    * 原子一致性：主要体现在事务提交前和提交后的状态仅有两种，不允许出现第三态
    * 并发一致性：四种隔离级别
* 外部一致性（External Consistency）
  * 核心是“并发控制”。
  * 相关概念是CAP
  * 主要问题是，新的值不能被立即读到
  * 强一致性：指系统中的某个数据被成功更新后，后续任何对该数据的读取操作都将得到更新后的值
  * 弱一致性：弱一致性是相对于强一致性而言，它不保证总能得到最新的值；
  * 最终一致性：是弱一致性的特殊形式，即保证在没有新的更新的条件下，经过一段“不一致时间窗口”，最终所有的访问都是最后更新的值。最常见的是DNS服务，更新域名指向的机器后，多级缓存要等到expiration time的时候才会更新，但是随着时间的推移，最终数据会趋于一致。

### Linearizability

Linearizability是分布式系统，并发编程领域的东西

两个事务T1,T2，T1先开始，更新数据对象o，T1提交。接着T2开始，读数据对象o，提交。以下两种调度：

1. T1,T2，满足Serializability，也满足Linearizability。
2. T2,T1，满足Serializability，不满足Linearizability，因为T1之前更新的数据T2读不到

* Spanner基于原子钟实现了Linearizability
* CockroachDB没有高精度原子钟，而是通过NTP同步时钟，机器之间时钟偏差一般能控制到250ms以内，但是也不绝对，这受到网络延时，系统load等因素的影响
* TiDB和Percolator一样，通过timestamp oracle提供唯一时钟源，同样实现了Linearizability

### OLTP和OLAP

* OLTP (Online Transactional Processing)
  * 联机事务处理：是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易
  * 后端程序员关注
* OLAP (Online Analytical Processing)
  * 是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。
  * 数据分析师关注

### NewSQL 分布式数据库

* 无妥协地支持 SQL，支持 JOIN / GROUP BY / 子查询等复杂查询和一定的大数据分析能力
* 支持 ACID 事务，支持强隔离级别，至少需要能做到可重复读隔离级别
* 具有弹性伸缩的能力，扩容缩容对于业务层完全透明，只需要简单的增减机器
* 真正的高可用能力，异地多活、故障恢复的过程不需要人为的接入，系统能够自动地容灾和进行强一致的数据恢复

## 什么时候使用分布式数据库？

### 分布式系统不是银弹

其实并没有什么技术是完美和包治百病的，在存储领域更是如此，

* 如果你的数据能够在一个 MySQL 装下并且服务器的压力不大，
* 或者对复杂查询性能要求不高，

那么其实分布式数据库并不是一个特别好的选择。

选用分布式的架构就意味着引入额外的维护成本，而且这个成本对于特别小的业务来说是不太划算的，即使你说需要高可用的能力，那 MySQL 的主从复制 + GTID 的方案可能也基本够用，这不够的话，还有最近引入的 Group Replication。而且 MySQL 的社区足够庞大，你能 Google 找到几乎一切常见问题的答案。

反过来说在如下情况下使用分布式数据库是好的选择：

* 单一数据库无法满足存储要求（容量方面）
* 对复杂查询性能的性能要求高（性能要求）

## TiDB

> 参考 [pingcap](https://pingcap.com/docs-cn/)

### 特点

* 结合了传统的 RDBMS 和 NoSQL 的最佳特性
* 兼容 MySQL
* 支持无限的水平扩展，具备强一致性（Raft 算法）和高可用性
* 为 OLTP 和 OLAP 场景提供一站式的解决方案。

### 何时使用

* 使用单机MySql的项目，业务激增，导致单机无法负载。
* 新项目的数据存储方面的预期单机无法满足，技术栈为MySql的项目

## CockRoachDB

### 特点

* 只能使用SSL的方式进行连接，权限验证则使用常见的用户名以及密码。
* 只支持 PostgreSQL的协议，不支持 MySQL 协议
* 支持分布式事务的ACID的分布式数据，
* 支持ANSI SQL的最高隔离级别Serializability。
