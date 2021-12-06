---
title: CompletableFuture

description: 本篇文章将介绍：Future 的方法及使用示例
categories:
 - 多线程编程
---

> 不积跬步，无以至千里。不积小流，无以成江海。

## Future

引用：《线程池方法 Submit 与 Execute 的区别》文章。

execute 接口是无返回值的，与之相对应的是一个有返回值的接口Future submit。

## get() 和 complete(T value)

**get()：**
方法会阻塞在那，直到结果返回。

**complete(T value)：**
所有阻塞在get（）方法的线程都将获得返回结果。

示例：
```
public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
    ThreadPoolExecutor pool = new ThreadPoolExecutor(10,10,3000,TimeUnit.MILLISECONDS,new ArrayBlockingQueue<>(10));

    CompletableFuture<String> completableFuture = new CompletableFuture<>();
    pool.execute(()->{
        try {
            System.out.println("completableFuture.get() 调用开始");
            System.out.println(completableFuture.get());
            System.out.println("completableFuture.get() 调用结束");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    });
    pool.execute(()->{
        System.out.println("completableFuture.complete() 调用开始");
        completableFuture.complete("martin");
    });

}
```

结果：
```
completableFuture.get() 调用开始
completableFuture.complete() 调用开始
martin
completableFuture.get() 调用结束
```

## 提交任务：runAsync 与 supplyAsync

**runAsync(Runnable)：**
无返回值。

示例：
```
CompletableFuture<Void> voidCompletableFuture = CompletableFuture.runAsync(() -> {
    try {
        System.out.println("start");
        TimeUnit.SECONDS.sleep(3);
        System.out.println("end");
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});
System.out.println("get() 前代码开始执行");
voidCompletableFuture.get();
System.out.println("get() 后代码开始执行");
```

结果：
```
get() 前代码开始执行
start
end
get() 后代码开始执行
```

**supplyAsync(Supplier)：**
有返回值。

示例：
```
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(new Supplier<String>() {
    @Override
    public String get() {
        try {
            System.out.println("start");
            TimeUnit.SECONDS.sleep(3);
            System.out.println("end");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "Martin";
    }
});
System.out.println(completableFuture.get());
```

结果：
```
start
end
Martin
```

## thenRun、thenAccept 和 thenApply

**thenRun(Runnable)：**
后面是一个无参数、无返回值的方法，最终的返回值是CompletableFuture<Void>。
```
CompletableFuture<Void> voidCompletableFuture = CompletableFuture.supplyAsync(() -> {
    // ...
    return "Martin";
}).thenRun(() -> {
    // ...
});
```

**thenAccept(Consumer)：**
后面是一个有参数、无返回值的方法，最终的返回值是CompletableFuture<Void>。
```
CompletableFuture<Void> voidCompletableFuture = CompletableFuture.supplyAsync(() -> {
    return "Martin";
}).thenAccept((result) -> {
    System.out.println(result);
});
```

结果：
```
Martin
```

**thenApply(Function)：**
后面是一个有参数、有返回值的方法，最终的返回值是CompletableFuture<T>。
```
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
    // ...
    return "Martin";
}).thenApply((result) -> {
    return result + " Hua";
});
System.out.println(completableFuture.get());
```

结果：
```
Martin Hua
```

## allOf 和 anyOf

**allOf：**
等待所有 CompletableFuture 结束之后再继续做下面的事，返回值是 CompletableFuture<Void>。

**anyOf：**
只要有一个 CompletableFuture 结束就可以做下面的事，返回值是 CompletableFuture<Object>。