---
title: Stream 项目 - Seata
description: 本篇文章将介绍：选用 seata 的原因、集成过程、探坑
categories:
 - Architectural Experience
---

> 学海无涯，心存高远。

## 选用 seata 的原因

在对分布式事务组件进行选型时，也挑选了几种解决方案：

1. TX-LCN
2. Seata
3. MQ异步通知

TX-LCN：小团队维护，以往使用过程中多数据库组合事务有兼容问题。

Seata：迭代速度快，文档匮乏，可以和 nacos 进行结合。

MQ 异步通知：性能较低，投入成本较高。

## 集成过程

这里不得不吐槽一下 Seata，文档匮乏到几乎可以说没有。整个集成过程充满了心酸，需要不停谷歌百度、看源码。

首先需要把 Seata server 搭建起来，这个过程中和运维一起翻阅 K8s 部署手册，就那么一点点，需要根据猜测和网上提供的帮助慢慢验证。

Seata server 启动好了之后，因为我客户端使用的 nacos 加载 seata 配置，所以需要将客户端的配置初始化到 nacos 中。

最后业务项目配置 Seata registry 和 Seata config。

## 探坑

**Seata 官方 K8s 部署手册不齐全**

谷歌、百度、推理

**本地 Docker 部署 ip 无法访问**

这个情况发生在用 Docker 搭建本地环境时遇到。docker 会自动分配容器 ip，不做处理时注册到 nacos 上的 Seata Server 地址是容器中的地址，本地是无法访问的。所以在将 Seata 注册到 nacos 的时候需要指定注册到 nacos 的 ip。（这不是 Seata 的问题，但需要注意一下）

**Seata 配置导入 nacos**

其中这个配置需要留意，“xxx_group”可以自定义，但需要跟 Seata 客户端（业务项目）中的“seata.tx-service-group”配置的值保持一致。

```
service.vgroupMapping.xxx_group=default
```
**不使用 nacos 随机生成的 namespace**

Seata config 需要指定 namespace，nacos 默认是会随机 namespace 的，从环境配置的角度考虑可以手动设置 namespace，省的每个环境都不一样。例如：

```
seata.config.nacos.namespace=seata
```

**版本号**

集成的时候也要对自己有信息，不一定就是有地方没配对，也有可能是版本冲突。可能是 alibaba 这一套迭代较快，改动较大，所以会出现 nacos 包、seata 包、sentinel 包互相冲突的情况。建议百度以下他们的版本对应关系。