---
title: ThreadPoolExecutor常用Api
description: 本篇文章将介绍：ThreadPoolExecutor类的常用方法（shutdown()、shutdownNow()、awaitTermination(long timeout,TimeUnit unit)、execute(Runnable command)...）
categories:
 - 多线程编程
---

> 不积跬步，无以至千里。不积小流，无以成江海。

<style>
table th:first-of-type {
    width: 25%;
}
table th:nth-of-type(2) {
    width: 15%;
}
table th:nth-of-type(3) {
    width: 60%;
}
</style>

API | 返回 | 描述
---|---|---
shutdown() | void | 拒绝接收新任务，已经接收的任务会继续执行。
shutdownNow() | List<Runnable> | 尝试停止所有正在执行的任务，暂停正在等待的任务，并返回正在等待执行的任务的列表。
isShutdown() | boolean | 调用shutdown()或shutdownNow()方法后返回true。
isTerminated() | boolean | 调用shutdown()或shutdownNow()方法后，成功关闭后返回true。
isterminating() | boolean | 调用shutdown()或shutdownNow()方法后，如果正在终止但尚未完成，则返回 true。
awaitTermination(long timeout,TimeUnit unit) | boolean | 调用shutdown()或shutdownNow()方法后，等待时间内所有任务执行结束且线程池已关闭返回true，超时返回false，等待时被打断抛出InterruptedException。
execute(Runnable command) | void | 在将来的某个时间执行给定的任务。
getActiveCount() | int | 返回正在主动执行任务的线程的大概数量。
getCompletedTaskCount() | long | 返回已完成执行的任务的大概总数。
getCorePoolSize() | int | 返回线程的核心数量。
getKeepAliveTime(TimeUnit unit) | long | 根据时间单位返回线程保持活动时间。
getLargestPoolSize() | int | 返回池中曾经同时存在的最大线程数。
getMaximumPoolSize() | int | 返回允许的最大线程数。
getPoolSize() | int | 返回池中的当前线程数。
getQueue() | BlockingQueue<Runnable> | 返回此执行程序使用的任务队列。
getRejectedExecution-Handler() | RejectedExecutionHandler | 返回无法执行任务的当前处理程序。
getTaskCount() | long | 返回计划执行的任务总数。
getThreadFactory() | ThreadFactory | 返回用于创建新线程的线程工厂。
prestartAllCoreThreads() | int | 启动所有核心线程，使它们空闲地等待工作。
prestartCoreThread() | boolean | 启动一个核心线程，使其闲置地等待工作。
setCorePoolSize(int corePoolSize) | void | 设置核心线程数。
setKeepAliveTime(long time, TimeUnit unit) | void | 设置线程在终止之前可能保持空闲的时间限制。
setMaximumPoolSize(int maximumPoolSize)| void | 设置允许的最大线程数。
setRejectedExecution-Handler(RejectedExecutionHandler handler) | void | 为无法执行的任务设置新的处理程序。
setThreadFactory(ThreadFactory threadFactory) | void | 设置用于创建新线程的线程工厂。
