---
title: 线程池方法 Submit 与 Execute 的区别
description: 本篇文章将介绍：Submit 与 Execute 方法接收的任务类型、返回值、异常之间的区别，以及造成这一区别的原因（Runnable 与 Callable）
categories:
 - 多线程编程
---

> 不积跬步，无以至千里。不积小流，无以成江海。

## submit 与 execute区别


```
源码：
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```

```
源码：
public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
}

public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}

public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
```

**任务类型：**
execute 只能接受 Runnable 类型的任务，submit 可以接受 Runnable、Callable 类型的任务。

**返回值：**
execute 没有返回值，submit 有返回值。但接受 Runnable 任务时返回值均为 void，所以使用 Future 的 get() 获得的还是 null。

**异常：**
execute 中的是 Runnable 接口的实现，所以只能使用 try、catch 来捕获 CheckedException，通过实现 UncaughtExceptionHande 接口处理UncheckedException。

submit 中不管提交的是 Runnable 还是 Callable 类型的任务，如果不对返回值 Future 调用 get() 方法，都会吃掉异常。

## Runnable 与 Callable 区别

```
源码：
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

```
源码：
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

Callable 拥有返回值且能够抛出 Exception 异常，所以不管是 CheckedException 还是 UncheckedException，直接抛出即可。下面示例演示了如何创建并执行Callable类型的任务。


```
示例：
// 创建 Callable 实例
class CallableTask implements Callable<Boolean>{
    public Boolean call() throws Exception {
        return false;
    }
} 

// 执行 Callable 实例
Future<Boolean> future = executor.submit(new CallableTask());
try {
    future.get();
} catch (InterruptedException e) {
    e.printStackTrace();
} catch (ExecutionException e) {
    e.printStackTrace();
}
```
