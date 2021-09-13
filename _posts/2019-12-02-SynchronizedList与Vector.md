---
title: SynchronizedList与Vector
description: 本篇文章将介绍：线程安全的集合SynchronizedList与Vector，以及使用示例
categories:
 - 多线程编程
---

> 不积跬步，无以至千里。不积小流，无以成江海。

## SynchronizedList与Vector区别
1、Vector扩容为原来的2倍长度，ArrayList扩容为原来1.5倍。

2、SynchronizedList有很好的扩展和兼容功能。他可以将所有的List的子类转成线程安全的类。

3、使用SynchronizedList的时候，进行遍历时要手动进行同步处理。

4、SynchronizedList可以指定锁定的对象。

**关于第三点的详细说明：**

```
官方示例：
List list = Collections.synchronizedList(new ArrayList());
      ...
  synchronized (list) {
      Iterator i = list.iterator(); // Must be in synchronized block
      while (i.hasNext())
          foo(i.next());
  }
```
打开源码可以看到，Collections.synchronizedList并没有对Iterator<E> iterator();方法加锁，所以该方法不是线程安全的，如果要遍历，还是必须要在外面加一层锁。

## 示例

```
示例：
public static void main(String[] args) throws InterruptedException {
    ThreadPoolExecutor threadPool = new ThreadPoolExecutor(20, 20, 3,
            TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(200)
    );

    Vector<Integer> vector = new Vector(2000);
    List synchronizedList = Collections.synchronizedList(new ArrayList(2000));
    List<Integer> list = new ArrayList<>(2000);
    for (int i=0; i<2000; i++){
        int finalI = i;
        threadPool.submit(()->{
            vector.add(finalI);
            synchronizedList.add(finalI);
            list.add(finalI);
        });
    }

    threadPool.shutdown();
    if (threadPool.awaitTermination(5,TimeUnit.SECONDS)){
        System.out.println("vector.size():" + vector.size());
        System.out.println("synchronizedList.size():" + synchronizedList.size());
        System.out.println("list.size():" + list.size());
    }else{
        System.out.println("超时");
    }
}
    
输出：
vector.size():2000
synchronizedList.size():2000
list.size():1999
```

上述示例中演示了使用线程不安全的集合可能带来的后果。尽管它不是百分百发生，但在多线程中应当使用线程安全的对象或者使用排它锁。
