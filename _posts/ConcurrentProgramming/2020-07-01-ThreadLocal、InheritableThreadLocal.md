---
title: ThreadLocal、InheritableThreadLocal
description: 本篇文章将介绍：ThreadLocal、InheritableThreadLocal获取线程属性
categories:
 - Concurrent Programming
---

> 不积跬步，无以至千里。不积小流，无以成江海。

## 类ThreadLocal

类ThreadLocal主要解决的就是每个线程绑定自己的值，可以将ThreadLocal类比喻成全局存放数据的盒子，盒子中可以存储每个线程的私有数据（每个线程使用同一个ThreadLocal存放值，线程之间是隔离的）。

**方法get() 与 null**

示例：
```
public static ThreadLocal t1 = new ThreadLocal();
public static void main(String[] args){
    if(t1.get() == null){
        System.out.println("没有放值");
        t1.set("111");
    }
    System.out.println(t1.get());
}
```
运行结果如下：

```
没有放值
111
```
从上述示例结果可以看出，第一遍使用get()方法获取数据的时候结果为null。

**解决get()返回null问题**

```
public class ThreadLocalExt extends ThreadLocal {
    @Override
    protected Object initialValue(){
        return "初始值";
    }
}
```

## 类InheritableThreadLocal
类InheritableThreadLocal可以在子线程中取得父线程继承下来的值。

示例：

```
public class InheritableThreadLocalExt extends InheritableThreadLocal{
    @Override
    protected Object initialValue(){
        return "父值";
    }
}

public class Tools {
    public static InheritableThreadLocalExt t1 = new InheritableThreadLocalExt();
}

子线程：
public class ThreadA extends Thread {
    @Override
    public void run(){
        System.out.println(Tools.t1.get());
    }
}

父线程：
public static void main(String[] args){
    System.out.println(Tools.t1.get());
    ThreadA a = new ThreadA();
    a.start();
}
```
结果：

```
父值
父值
```

**继承值并修改**

如果需要对父类中继承来的值加以修改，需要对InheritableThreadLocalExt类进行以下修改：


```
public class InheritableThreadLocalExt extends InheritableThreadLocal{
    @Override
    protected Object initialValue(){
        return "父值";
    }
    @Override
    protected Object childValue(Object parentValue){
        return parentValue + ",子值";
    }
}
```
结果：

```
父值
父值,子值
```

但在使用InheritableThreadLocal类需要注意一点的是，如果子线程在取得值的同时，主线程将InheritableThreadLocal中的值进行更改，那么子线程取到的值还是旧值。
