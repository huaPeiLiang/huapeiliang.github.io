---
title: Stream项目-Nacos
description: 本篇文章将介绍：选用nacos的原因、集成过程
categories:
 - 架构之路
---

> 学海无涯，心存高远。

## 选用nacos的原因

选型时比较了目前常见的解决服务注册与发现的几个组件：

- Eureka
- Zookeeper
- Nacos

比较下来最终使用了nacos，因为他有较好的可视化界面；alibaba cloud解决方案的一员和其他组件更好的配合；有服务元数据、流量控制、分组等拓展功能。

## 集成过程

整个过程中没有什么大坑。但是版本号一定要选择好，因为后续引入seata、sentinel等都会有影响。

**配置存储**

nacos内置了数据库，直接启动nacos  server就可以使用了。但在项目中我选择了搭一个mysql数据库，将配置存到自己的数据库中。方便查询和备份。

**配置分组**

nacos配置可以根据namespace、group进行两个维度的分类。项目中可以进行如下划分：

- 将namespace根据场景划分为：public（业务）、seata（事务）、sentinel（限流熔断）。

- 将group根据环境划分为：dev（开发）、sit（测试）、pre（验证）、prd（生成）。

**元数据**

项目中元数据配合gateway进行金丝雀发布。在元数据里可以自定义版本、权重，gateway根据版本和权重选择请求分发到哪里。