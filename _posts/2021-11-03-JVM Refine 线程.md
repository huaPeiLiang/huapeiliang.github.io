---
title: JVM Refine 线程
description: 本篇文章将介绍：Refine 线程简介、抽样线程、管理 Rset、Mutator 处理 DCQ
categories:
- JVM
---

> 君子以自强不息。

## Refine 线程简介

Refine 线程默认数目为 G1ConcRefinementThreads+1。主要功能：

- 用于处理新生代分区的抽样，并且在满足响应时间的这个指标下，更新 YHR 的数目。通常有一个线程来处理。
- 管理 RSet，这是 Refine 最主要的功能。RSet 的更新并不是同步完成的，G1 会把所有的引用关系都先放入到一个队列中，称为 dirty card queue（DCQ），然后使用线程来消费这个队列以完成更新。正常来说有 G1ConcRefinementThreads 个线程处理；实际上除了 Refine 线程更新 RSet 之外，GC 线程或者 Mutator 也可能会更新 RSet；DCQ 通过 Dirty Card Queue Set（DCQS）来管理；为了能够并发地处理，每个 Refine 线程只负责 DCQS 中的某几个 DCQ。

## 抽样线程

Refine 线程池中的最后一个线程就是抽样线程，它的主要作用是设置新生代分区的个数，使 G1 满足垃圾回收的预测停顿时间。

## 管理 Rset

JVM 在设计的时候，声明了一个全局的静态变量 DirtyCardQueueSet（DCQS），DCQS 里面存放的是 DCQ，为了性能的考虑，所有处理引用关系的线程共享一个 DCQS，每个 Mutator（线程）在初始化的时候都关联这个 DCQS。每个 Mutator 都有一个私有的队列，每个队列的最大长度由 G1UpdateBufferSize（默认值为256）确定，即最多存放256个引用关系对象，在本线程中如果产生新的对象引用关系则把引用者放入 DCQ 中，当满256个时，就会把这个队列放入到 DCQS 中（DCQS 可以被所有线程共享，所以放入时需要加锁），当然可以手动提交当前线程的队列（当队列还没有满的时候，提交时要指明有多少个引用关系）。而 DCQ 的处理则是通过 Refine 线程。

## Mutator 处理 DCQ

队列 set 的最大长度依赖于 Refine 线程的个数，最大为 Red Zone 的个数。当队列 set 里面的队列个数超过 Red Zone 的个数时，提交队列的 Mutator 就不能把这个队列放入到 set 中，此时，Mutator 就会直接处理这个队列的引用。
