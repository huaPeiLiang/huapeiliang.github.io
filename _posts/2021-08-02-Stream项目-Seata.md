---
title: Stream项目-Seata
description: 本篇文章将介绍：选用seata的原因、集成过程、探坑
categories:
 - 架构之路
---

> 学海无涯，心存高远。

## 选用seata的原因

在对分布式事务组件进行选型时，也挑选了几种解决方案：

1. TX-LCN
2. Seata
3. MQ异步通知

TX-LCN：小团队维护，以往使用过程中多数据库组合事务有bug。

Seata：迭代速度快，文档匮乏，可以和nacos进行结合。

MQ异步通知：性能较低，投入成本较高。

## 集成过程

这里不得不吐槽一下Seata，文档匮乏到几乎可以说没有。整个集成过程充满了心酸，需要不停谷歌百度、看源码。

首先需要把Seata server搭建起来，这个过程中和运维一起翻阅K8s部署手册，就那么一点点，需要根据猜测和网上提供的帮助慢慢验证。

Seata server启动好了之后，因为我客户端使用的nacos加载seata配置，所以需要将客户端的配置初始化到nacos中。

最后业务项目配置Seata registry和Seata config。

## 探坑

**Seata 官方K8s部署手册不齐全**

谷歌、百度、推理

**本地Docker部署ip无法访问**

这个情况发生在用Docker搭建本地环境时遇到。docker会自动分配容器ip，不做处理时注册到nacos上的Seata Server地址是容器中的地址，本地是无法访问的。所以在将Seata注册到nacos的时候需要指定注册到nacos的ip。（这不是Seata的问题，但需要注意一下）

**Seata配置导入nacos**

其中这个配置需要留意，“xxx_group”可以自定义，但需要跟Seata客户端（业务项目）中的“seata.tx-service-group”配置的值保持一致。

```
service.vgroupMapping.xxx_group=default
```
**不使用nacos随机生成的namespace**

Seata config需要指定namespace，nacos默认是会随机namespace的，从环境配置的角度考虑可以手动设置namespace，省的每个环境都不一样。例如：

```
seata.config.nacos.namespace=seata
```

**版本号**

集成的时候也要对自己有信息，不一定就是有地方没配对，也有可能是版本冲突。可能是alibaba这一套迭代较快，改动较大，所以会出现nacos包、seata包、sentinel包互相冲突的情况。建议百度以下他们的版本对应关系。