---
title: Synchronized 关键字
description: 本篇文章将介绍：锁的实现分类、锁的几个概念（可重入性、锁的征用与调度、锁的粒度）
categories:
 - 多线程编程
---

> 不积跬步，无以至千里。不积小流，无以成江海。

## synchronized 历史

从 JDK5 引入了现代操作系统新增加的 CAS 原子操作（ JDK5 中并没有对 synchronized 关键字做优化，而是体现在 J.U.C 中，所以在该版本 concurrent 包有更好的性能 ），
从 JDK6 开始，就对 synchronized 的实现机制进行了较大调整，包括使用 JDK5 引进的 CAS 自旋之外，还增加了自适应的 CAS 自旋、锁消除、锁粗化、偏向锁、轻量级锁这些优化策略。

## synchronized 错误示例
下面代码会带来一个问题，锁失效了。
```
public static void main(String[] args) throws InterruptedException {
    ThreadPoolExecutor threadPool = new ThreadPoolExecutor(20, 20, 3,
            TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(1000)
    );

    MyTask myTask = new MyTask();
    for (int i=0; i<1000; i++){
        threadPool.execute(myTask);
    }

    threadPool.shutdown();
    if (threadPool.awaitTermination(5,TimeUnit.SECONDS)){
        myTask.getResult();
    }else{
        System.out.println("超时");
    }
}


// 自定义任务处理
class MyTask implements Runnable{
    private Integer num = 0;
    private List<Integer> numList = new ArrayList<>(1000);

    @Override
    public void run() {
        synchronized (num){
            Integer numCopy = num;
            num = numCopy + 1;
            numList.add(num);
        }
    }

    public void getResult(){
        HashSet<Integer> numSet = new HashSet<>(numList);
        System.out.println("numList.size()1:" + numList.size());
        System.out.println("numSet.size()1:" + numSet.size());
    }
}

输出：
numList.size()1:999
numSet.size()1:996
```
**synchronized(锁句柄){}，锁句柄的变量通常使用 final 修饰。这是因为锁句柄的的值一旦改变，会导致执行同一个同步块（synchronized）的多个线程实际上使用的不是同一把锁。**
这里可以改为 synchronized(this){}。