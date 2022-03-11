---
title: 线程池参数 WorkQueue
description: 本篇文章将介绍：ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue、PriorityBlockingQueue
categories:
 - Concurrent Programming
---

> 不积跬步，无以至千里。不积小流，无以成江海。

## ArrayBlockingQueue（有界队列）
必须为其设置容量，当容量设置为 Integer.MAX_VALUE 时和 new LinkedBlockingQueue(Integer.MAX_VALUE) 效果一样。排队策略为 FIFO（先进先出）。


## LinkedBlockingQueue（无界队列）
可以不为其设置容量，当不设置容量时默认为 Integer.MAX_VALUE。排队策略为 FIFO（先进先出）。

## SynchronousQueue（直接提交）
该队列没有容量。提交的任务不会被真实的保存在队列中，而总是将新任务提交给线程执行。如果没有空闲的线程，则尝试创建新的线程。如果线程数大于最大值 maximumPoolSize，则执行拒绝策略。比如：现在有一个核心线程数为2，最大线程数为3，队列策略使用的 SynchronousQueue 的线程池。在两个核心线程都在执行任务的同时又来了一个任务A，这时会创建一个新的线程执行任务A。在任务A以及核心线程数都在工作时，又来了一个任务B，这是最大线程数已满，任务B会直接执行拒绝策略。

## PriorityBlockingQueue（优先任务队列）
它是一种特殊的无界队列。和 FIFO（先进先出）的策略有所不同，PriorityBlockingQueue 会按照任务自身的优先级顺序先后执行（总是确保高优先级的任务先执行）。

## 总结
当使用无界队列且没有明确指定容量或使用容量为 Integer.MAX_VALUE 的有界队列时，当核心线程都在工作且执行时间很长的情况下，会导致队列无限扩大（maximumPoolSize失效），直到系统资源耗尽。**