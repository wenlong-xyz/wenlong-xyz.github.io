---
title: 阿里巴巴中间件性能挑战赛-设计简介
category: Technology
toc: true
date: 2017-03-15 20:04:20
tags:
---
&emsp;&emsp;先放几张中间件比赛时的设计图，文字内容会后续补充

# 需求描述
&emsp;&emsp;4G商品和买家数据、100G订单数据（4亿订单）。三张表的关系如下图，要求实现的功能是：实现订单的快速查询功能，包括：数据的单条查询、范围查询、多表联合查询。
* 根据订单ID查询订单数据（可能会设计商品的信息）
* 查询某个用户在某一段时间内的交易信息
* 查询某个商品涉及的交易信息
![](1-demand-analysis.png)

# 索引设计
## 索引示例，根据订单ID查询订单信息
![](2-index-main.png)
## 其他索引，应对范围查询
![](2-index-other.png)

# 优化设计
## 订单索引数据间隔缓存
![](3-optimize-order.png)
## 订单数据按照商品ID聚集
![](3-optimize-goodorder.png)


# 非中间件比赛内容-CNHeatmaps设计图
## 整体架构
![](5-cnheatmap-1.png)
## 前端解析流程
![](5-cnheatmap-2.png)
