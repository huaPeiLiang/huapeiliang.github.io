---
title: ForkJoinPool状态控制

description: 本篇文章将介绍：状态变量ctl解析及它的五个部分、阻塞栈–Treiber Stack、ctl变量的初始值、ForkJoinWorkThread状态与个数分析
categories:
 - 多线程编程
---

> 不积跬步，无以至千里。不积小流，无以成江海。

## 状态变量ctl解析

类似于ThreadPoolExecutor，在ForkJoinPool中也有一个ctl变量负责表达ForkJoinPool的整个生命周期和相关的各种状态。不过ctl变量更加复杂，是一个long型变量。

ctl是一个64个比特位的变量，被分成以下五部分：

**AC：** 最高的16个比特位，表示Active线程数-parallelism，parallelism是上面的构造函数传进去的参数。

**TC：** 次高的16个比特位，表示Total线程数-parallelism。

**ST：** 1个比特位，如果是1，表示整个ForkJoinPool正在关闭。

**EC：** 15个比特位，表示阻塞栈的栈顶线程的wait count。

**ID：** 16个比特位，表示阻塞栈的栈顶线程对应的poolIndex。

## 阻塞栈–Treiber Stack

要实现多个线程的阻塞、唤醒，除了park/unpark这一对操作原语，还需要一个无锁链表实现的阻塞队列，把所有阻塞的线程串在一起。

在ForkJoinPool中，没有使用阻塞队列，而是使用了阻塞栈。把所有空闲的Worker线程放在一个栈里面，这个栈同样通过链表来实现，名为Treiber Stack。

首先，ForkJoinWorkerThread有一个poolIndex变量，记录了自己在ForkJoinWorkerThread[]数组中的下标位置，poolIndex变量就相当于每个ForkJoinPoolWorkerThread对象的地址；其次，ForkJoinWorkerThread还有一个nextWait变量，记录了前一个阻塞线程的poolIndex，这个nextWait变量就相当于链表的next指针，把所有的阻塞线程串联在一起，组成一个Treiber Stack。

最后，ctl变量的最低16位，记录了栈的栈顶线程的poolIndex；中间的15位，记录了栈顶线程被阻塞的次数，也称为wait count。

## ctl变量的初始值

初始的ForkJoinPool 中的线程个数为0，所以AC=0-parallelism，TC=0-parallelism。这意味着只有高32位的AC、TC 两个部分填充了值，低32位都是0填充。

## ForkJoinWorkThread状态与个数分析

在ThreadPoolExecutor中，有corePoolSize和maxmiumPoolSize 两个参数联合控制总的线程数。

而在ForkJoinPool中只传入了一个parallelism参数，且这个参数并不是实际的线程数。通过crl和blockedCount这两个变量，可以知道在整个ForkJoinPool中，所有空闲线程、活跃线程以及阻塞线程的数量。

**ForkJoinPool中的线程都可能有哪几种状态**

- 空闲状态（放在Treiber Stack里面）。

- 活跃状态（正在执行某个ForkJoinTask，未阻塞）。

- 阻塞状态（正在执行某个ForkJoinTask，但阻塞了，于是调用join，等待另外一个任务的结果返回）。

**ctl变量如何反映这三种状态**

高32位：u=（int）（ctl＞＞＞32），然后u又拆分成tc、ac 两个16位；

低32位：e=（int）ctl。

- e＞0，说明Treiber Stack不为空，有空闲线程；e=0，说明没有空闲线程。

- ac＞0，说明有活跃线程；ac＜=0，说明没有空闲线程，并且还未超出parallelism。

- tc＞0，说明总线程数＞parallelism。