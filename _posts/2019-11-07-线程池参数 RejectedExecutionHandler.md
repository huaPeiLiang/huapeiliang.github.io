---
title: 线程池参数 RejectedExecutionHandler
description: 本篇文章将介绍：AbortPolicy、DiscardPolicy、DiscardOldestPolicy、CallerRunsPolicy、自定义拒绝策略
categories:
 - 多线程编程
---

> 不积跬步，无以至千里。不积小流，无以成江海。

## AbortPolicy（默认策略）
该策略是线程池的默认策略。使用该策略时，如果线程池队列满了丢掉这个任务并且抛出 RejectedExecutionException 异常。

```
源码：
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r.toString() +
                                                 " rejected from " +
                                                 e.toString());
        }
```


## DiscardPolicy
如果线程池队列满了，会直接丢掉这个任务并且不会有任何异常。

```
源码：
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            // 空方法
        }
```


## DiscardOldestPolicy
如果队列满了，会将最早进入队列的任务删掉腾出空间，再尝试加入队列。

```
源码：
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                e.getQueue().poll();
                e.execute(r);
            }
        }
```

## CallerRunsPolicy
如果添加到线程池失败，那么主线程会自己去执行该任务，不会等待线程池中的线程去执行。

```
源码：
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                r.run();
            }
        }
```

## 自定义拒绝策略
如果以上策略都不符合业务场景，那么可以自己定义一个拒绝策略，只要实现 RejectedExecutionHandler 接口，并且实现 rejectedExecution 方法就可以了。具体的逻辑就在 rejectedExecution 方法里去定义就OK了。

```
示例：
class MyRejectPolicy implements RejectedExecutionHandler{
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        // todo
    }
}
```