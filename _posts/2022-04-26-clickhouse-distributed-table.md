---
title: "ClickHouse之Distributed Table Engine"
categories:
  - Blog
tags:
  - db
  - clickhouse
  - 分布式引擎
---
# Clickhouse是什么？
Clickhouse是一个面向列的数据库管理系统(DBMS)，用于在线查询分析处理(OLAP),它支持多种database engine和table engine，我们可以在不同的业务场景下选择合适的table engine。

# 分布式表引擎
表引擎众多，可能你不会用到所有的类型，但有一种引擎是迟早会用到的，它就是分布式引擎，因为它是当table中数据达到一定量级时，进行水平扩展的主要方式。

*分布式引擎本身不存储数据*, 但可以在多个服务器上进行分布式查询。
## 读取操作
将查询请求转发到多个远端服务器进行并行查询，然后返回合并后的查询结果。

## 写入操作
- 直接将请求发送到存储数据的db
- 将请求发送到分布式表，再由分布式表所在服务器将请求分配到不同的数据存储服务器
