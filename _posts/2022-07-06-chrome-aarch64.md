---
title: "Build chromium for your aarch64 Linux"
categories:
  - 坑
tags:
  - web
  - caddy
  - https
---
# chromium是什么？
Clickhouse是一个面向列的数据库管理系统(DBMS)，用于在线查询分析处理(OLAP)，单机查询速度大于现有任何数据库，它支持多种database engine[^4]和table engine，不同的table engine[^5]提供不同的特性，以满足不同的业务场景。

# 分布式表引擎
表引擎众多，可能你不会用到所有的类型，但有一种引擎是迟早会用到的，它就是分布式引擎，因为它是当table中数据达到一定量级时，进行水平扩展的主要方式。
*分布式引擎本身不存储数据*, 但可以在cluster中的多个服务器上进行分布式并发查询。cluster通过配置文件来定义.
分布式表创建语句如下:
```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = Distributed(cluster, database, table[, sharding_key[^6][, policy_name]])
[SETTINGS name=value, ...]
```


> 参考
> > https://github.com/chromium/chromium/blob/main/build/install-build-deps.sh
> > https://github.com/allen12921/allen12921.github.io/edit/master/_posts/2022-07-06-chrome-aarch64.md
<script src="{{ "/assets/js/mermaid.min.js" | relative_url }}"></script>
