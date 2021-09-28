---
title: ForkJoinPool的关闭

description: 本篇文章将介绍：关键的terminate变量、shutdown（）与 shutdownNow（）
categories:
 - 多线程编程
---

> 不积跬步，无以至千里。不积小流，无以成江海。


同ThreadPoolExecutor 一样，ForkJoinPool 的关闭也不可能是“瞬时的”，而是需要一个平滑的过渡过程。

## 关键的terminate变量

对于一个Worker线程来说，它会在一个while循环里面不断轮询队列中的任务，如果有任务，那么执行，处在活跃状态；如果没有任务，则进入空闲等待状态。

（int）（c=ctl）＜0，即低32位的最高位为1（参考前面ctl变量的解析），说明线程池已经进入了关闭状态。但线程池进入关闭状态，不代表所有的线程都会立马关闭。

为此，在ForkJoinWorkerThread里还有一个terminate变量，初始为false。当线程池要关闭的时候，会把相关线程的terminate变量置为true。这样，这些线程就会退出上面的while循环，也就会自动退出。

## shutdown（）与 shutdownNow（）

```
    public void shutdown() {
        checkPermission();
        tryTerminate(false, true);
    }
```

```
    public List<Runnable> shutdownNow() {
        checkPermission();
        tryTerminate(true, true);
        return Collections.emptyList();
    }
```
调用tryTerminate（boolean）函数，其中一个传入的是false，另一个传入的是true。tryTerminate意为试图关闭ForkJoinPool，并不保证一定可以关闭成功。

```
    public boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (this == common) {
            awaitQuiescence(timeout, unit);
            return false;
        }
        long nanos = unit.toNanos(timeout);
        if (isTerminated())
            return true;
        if (nanos <= 0L)
            return false;
        long deadline = System.nanoTime() + nanos;
        synchronized (this) {
            for (;;) {
                if (isTerminated())
                    return true;
                if (nanos <= 0L)
                    return false;
                long millis = TimeUnit.NANOSECONDS.toMillis(nanos);
                wait(millis > 0L ? millis : 1L);
                nanos = deadline - System.nanoTime();
            }
        }
    }
```

同ThreadPoolExecutor一样，ForkJoinPool中也有awaitTermination（…）函数，代码几乎相同，上面函数的最后一段，就是在整个线程池都已关闭，即没有任何线程存活的情况下，通知阻塞在awaitTermination（..）的外部线程。

在startTerminating（）中，把全局队列、每个线程的局部队列中的任务都取消了，同时把所有线程的terminate置为了true，唤醒了阻塞栈中所有等待的空闲线程（这些线程的terminate置为了true，会自动退出）。

如果now为false，tryTerminate（…）会返回false。只是在最外面的函数shutdown（）里面，把shutdown置为了true。这样，新的任务提交会被拒绝，但现有的任务都会正常执行完成。