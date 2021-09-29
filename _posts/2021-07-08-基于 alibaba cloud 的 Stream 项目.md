---
title: 基于 alibaba cloud 的 Stream 项目
description: 本篇文章将介绍：Stream项目背景、技术栈、我的工作
categories:
 - 架构之路
---

> 学海无涯，心存高远。

## 项目背景

这是我回归 Bite investments 后的项目。起因是 Bite 开始了 Saas 平台的立项，希望打造一个全球另类投资的 Saas 平台，他被命名为"Stream"。

## 技术栈

整体使用的 spring cloud 微服务架构。注册中心、配置中心使用 nacos，服务间调用使用 feign，限流使用 sentinel，事务使用 seata，消息队列使用 rocketMQ，
任务调度使用 XXL-Job，缓存使用 redis，关系数据库使用 mysql，NoSQL 数据库使用 mongodb，搜索引擎使用 elasticSearch，调用链追踪使用 skywalking，
日志可视化使用 Grafana,网关使用 gateway，服务健康检查使用 actuator

## 我的工作

后台架构建设、后台开发规范、业务开发

## 期望

这个项目刚刚诞生，希望他可以健康成长。后面可能会写多篇文章来记录这个过程。