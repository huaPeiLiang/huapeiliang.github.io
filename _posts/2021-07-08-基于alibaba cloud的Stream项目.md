---
title: 基于alibaba cloud的Stream项目
description: 本篇文章将介绍：Stream项目背景、技术栈、我的工作
categories:
 - 架构之路
---

> 学海无涯，心存高远。

## 项目背景

这是我回归Bite investments后的项目。起因是Bite开始了Saas平台的立项，希望打造一个全球另类投资的Saas平台，他被命名为"Stream"。

## 技术栈

整体使用的spring cloud微服务架构。注册中心、配置中心使用nacos，服务间调用使用feign，限流使用sentinel，事务使用seata，消息队列使用rocketMQ，
任务调度使用XXL-Job，缓存使用redis，关系数据库使用mysql，NoSQL数据库使用mongodb，搜索引擎使用elasticSearch，调用链追踪使用skywalking，
日志可视化使用Grafana,网关使用gateway，服务健康检查使用actuator

## 我的工作

后台架构建设、后台开发规范、业务开发

## 期望

这个项目刚刚诞生，希望他可以健康成长。后面可能会写多篇文章来记录这个过程。