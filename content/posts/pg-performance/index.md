---
title: "Postgresql 慢查询设置及查看"
date: 2020-06-03T15:27:59+08:00
draft: true
---

PG 的慢查询可以通过设置启动参数或是修改配置文件完成

<!--more-->

## `log_min_duration_statement` 项目

> alter database baseone set log\_min\_duration\_statement=1000;

这项配置可以临时开启，不用重启数据库。之后查询时间超过 1000 毫秒的 SQL 会被输出到日志。

## `pg_stat_statements` 扩展

见： https://www.postgresql.org/docs/9.4/pgstatstatements.html

这个扩展可以提供 sql 的一些统计数据，比如平均值、最大值、总时间等等。开启这个选项会有性能损耗。

启动时带上参数 

> -c shared\_preload\_libraries='pg\_stat\_statements' -c pg\_stat\_statements.max=10000 -c pg\_stat\_statements.track=all

然后在数据库启用

> CREATE EXTENSION IF NOT EXISTS pg\_stat\_statements;


随后收集一段时间的使用数据，可以直接使用 sql 查询慢的查询

```SQL
SELECT calls, total_time,max_time, mean_time, rows, 100.0 * shared_blks_hit /nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent,query FROM pg_stat_statements ORDER BY mean_time DESC LIMIT 20;
```

## 后续工作

分析查询（explain）然后优化，建立索引等等。
