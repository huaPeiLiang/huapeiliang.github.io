---
title: 线程池参数 ThreadFactory
description: 本篇文章将介绍：ThreadFactory
categories:
 - 多线程编程
---

> 不积跬步，无以至千里。不积小流，无以成江海。

在面向对象编程的世界中，工厂模式（factory pattern）是一个被广泛使用的设计模式。它是一个创建模式，它的目的是开发一个类，这个类的使命是创建一个或多个类的对象。然后，当我们要创建一个类的一个对象时，我们使用这个工厂而不是使用 new 操作。

使用这个工厂，我们集中对象的创建，获取容易改变创建对象的类的优势，或我们创建这些对象的方式，容易限制创建对象的有限资源。比如，我们只能有一个类型的N个对象，就很容易产生关于对象创建的统计数据。

Java 提供 ThreadFactory 接口，用来实现一个 Thread 对象工厂。Java 并发 API 的一些高级工具，如执行者框架（Executor framework）或 Fork/Join 框架（Fork/Join framework），使用线程工厂创建线程。


```
源码：
public interface ThreadFactory {

    /**
     * Constructs a new {@code Thread}.  Implementations may also initialize
     * priority, name, daemon status, {@code ThreadGroup}, etc.
     *
     * @param r a runnable to be executed by new thread instance
     * @return constructed thread, or {@code null} if the request to
     *         create a thread is rejected
     */
    Thread newThread(Runnable r);
}
```
该接口的目的是定制一个线程，可以设置线程的优先级、名字、是否后台线程、状态等。

输入：继承 Runnable 接口的实例对象。

输出：定制的线程，如果被拒绝，则返回 null。
