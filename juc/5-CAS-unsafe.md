# 保证原子性的两种方式

多线程环境不使用原子类保证线程安全（基本数据类型）

## 以前：使用volatile读+synchronized写

```java
/**
 * @auther zzyy
 * @create 2020-04-15 10:41
 */
public class T3
{
    volatile int number = 0;
    //读取
    public int getNumber()
    {
        return number;
    }
    //写入加锁保证原子性
    public synchronized void setNumber()
    {
        number++;
    }
}
 
 
```



## 现在使用原子类来保证

```java
/**
 * @auther zzyy
 * @create 2020-04-15 10:41
 */
public class T3
{
    volatile int number = 0;
    //读取
    public int getNumber()
    {
        return number;
    }
    //写入加锁保证原子性
    public synchronized void setNumber()
    {
        number++;
    }
    //=================================
    AtomicInteger atomicInteger = new AtomicInteger();

    public int getAtomicInteger()
    {
        return atomicInteger.get();
    }

    public void setAtomicInteger()
    {
        atomicInteger.getAndIncrement();
    }


}
 
```



# CAS是什么？

## CAS说明

compare and swap的缩写，中文翻译成比较并交换,实现并发算法时常用到的一种技术。它包含三个操作数——内存位置、预期原值及更新值。
执行CAS操作的时候，将内存位置的值与预期原值比较：
如果相匹配，那么处理器会自动将该位置值更新为新值，
如果不匹配，处理器不做任何操作，多个线程同时执行CAS操作只有一个会成功。 

原理：

![image-20220320104523128](https://image.imxyu.cn/file/image-20220320104523128.png)



## 硬件级别的保证

![image-20220320104657450](https://image.imxyu.cn/file/image-20220320104657450.png)

> 没有加任何锁，是非阻塞的，但是会一直进行比较，这个循环比较的过程是会消耗cpu的

## CAS代码

```java
public class CASDemo
{
    public static void main(String[] args) throws InterruptedException
    {
        AtomicInteger atomicInteger = new AtomicInteger(5);

        System.out.println(atomicInteger.compareAndSet(5, 2020)+"\t"+atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(5, 1024)+"\t"+atomicInteger.get());
    }
}
 
```



## 源码解析

![image-20220320104746877](https://image.imxyu.cn/file/image-20220320104746877.png)

# Unsafe类

## Unsafe

![image-20220320104818349](https://image.imxyu.cn/file/image-20220320104818349.png)



## atomicInteger.getAndIncrement()

![image-20220320104839548](https://image.imxyu.cn/file/image-20220320104839548.png)



## 源码分析

```
AtomicInteger atomicInteger = new AtomicInteger(0);
此时会传入一个初始值，并且AtomicInteger对象的这个初始值会使用valatile 关键字包裹，保证了线程之间的可见性
当一个线程更新了值之后，因为valatile关键字另一个线程会去取到该值，不断的更新值比较相加直到成功
```

new AtomicInteger().getAndIncrement();

![image-20220320104939496](https://image.imxyu.cn/file/image-20220320104939496.png)



![image-20220320105001301](https://image.imxyu.cn/file/image-20220320105001301.png)



## 底层汇编

### native修饰的方法代表是底层方法

![image-20220320105048389](https://image.imxyu.cn/file/image-20220320105048389.png)



### cmpxchg 

![image-20220320105130642](https://image.imxyu.cn/file/image-20220320105130642.png)



在不同的操作系统下会调用不同的cmpxchg重载函数，阳哥本次用的是win10系统

这里注意一个线程写的时候在cpu指令上会有一个lock 指令，保证同一时间只有一个线程能写入

![image-20220320105413137](https://image.imxyu.cn/file/image-20220320105413137.png)

### 总结

你只需要记住：CAS是靠硬件实现的从而在硬件层面提升效率，最底层还是交给硬件来保证原子性和可见性
实现方式是基于硬件平台的汇编指令，在intel的CPU中(X86机器上)，使用的是汇编指令cmpxchg指令。 

核心思想就是：比较要更新变量的值V和预期值E（compare），相等才会将V的值设为新值N（swap）如果不相等自旋再来。



## 回杀volatile 原子性问题

之前说过volatile 是不保证原子性的

这段代码加不到20000，原因就是有很多次操作失效了

![image-20220320105618542](https://image.imxyu.cn/file/image-20220320105618542.png)



结合图分析：

![image-20220320105841425](https://image.imxyu.cn/file/image-20220320105841425.png)

A,B两个线程同时读主内存中的值，都读到了5，然后同时修改成了6 ，因为同时只能一个写入主内存中（volatile 中也有lock指令保证），此时当A写入主内存后，主内存值变成了6。此时线程中的变量值就失效了，需要重新去读主内存中的值，此时B要先去读，读到为6。 在这个过程中B加的那一次就失效了，此时B+1，变成了7， 中间有一次加的结果失效了。 因此i<=1000循环虽然加了1000次，但是中间有一些结果都被作废了。



而cas就不一样了

CAS会每次循环比较这个值，如果这个值不是预期值的话就不会进行操作，就不存在作废的情况。（是原子级别的cpu指令保证的）



# 原子引用

AtomicInteger原子整型，可否有其它原子类型？

订单类、书本类、任何对象都可以设置成原子的

```java
package com.atguigu.Interview.study.thread;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.ToString;

import java.util.concurrent.atomic.AtomicReference;

@Getter
@ToString
@AllArgsConstructor
class User
{
    String userName;
    int    age;
}

/**
 * @auther zzyy
 * @create 2018-12-31 17:22
 */
public class AtomicReferenceDemo
{
    public static void main(String[] args)
    {
        User z3 = new User("z3",24);
        User li4 = new User("li4",26);

        AtomicReference<User> atomicReferenceUser = new AtomicReference<>();

        atomicReferenceUser.set(z3);
        System.out.println(atomicReferenceUser.compareAndSet(z3,li4)+"\t"+atomicReferenceUser.get().toString());
        System.out.println(atomicReferenceUser.compareAndSet(z3,li4)+"\t"+atomicReferenceUser.get().toString());
    }
}

```



# 自旋锁（*）

 根据cas的启发，我们可以在修改的时候进行比较，不断循环的判断值，如果和期望值相等的话再修改

![image-20220320112023975](https://image.imxyu.cn/file/image-20220320112023975.png)



## 手写一个自旋锁（必会）

```java

public class SpinLockDemo {

    AtomicReference<Thread> atomicReference=new AtomicReference<>();

    public void lock(){
        //lock的时候希望是null(上厕所没有人占用)，那么我就可以修改成自己。一直循环等待
        while (!atomicReference.compareAndSet(null,Thread.currentThread())){

        }
        System.out.println(Thread.currentThread().getName()+"\t 进来了");
    }


    public void unLock(){
        //解锁的时候就不用循环判断了，如果是自己的话就释放锁
        atomicReference.compareAndSet(Thread.currentThread(),null);
        System.out.println(Thread.currentThread().getName()+"\t 释放锁");

    }
    public static void main(String[] args) {
        SpinLockDemo spinLockDemo=new SpinLockDemo();
        new Thread(() -> {
            spinLockDemo.lock();
            //第一个线程持有锁之后等5秒释放
            try {   TimeUnit.SECONDS.sleep(5);   } catch (InterruptedException e) { e.printStackTrace(); }
            spinLockDemo.unLock();
        }, "t1").start();
        //暂停一会儿线程，保证A线程先于B线程启动并完成
        try { TimeUnit.SECONDS.sleep( 1 ); } catch (InterruptedException e) { e.printStackTrace(); }

        //第二个线程想直接获取的时候被t1占用，会等待5秒t1释放完了之后
        new Thread(() -> {
            spinLockDemo.lock();
            spinLockDemo.unLock();
        }, "t2").start();
    }

}

```

![image-20220320114923814](https://image.imxyu.cn/file/image-20220320114923814.png)





# CAS缺点

## 循环时间长开销很大

![image-20220320134309015](https://image.imxyu.cn/file/image-20220320134309015.png)

这也是锁饥饿问题，这个线程每次修改都被其他线程修改过了，这个线程每次想提交的时候另一个线程都在他的前面提交过了，这就会导致这个线程一直抢不到资源，锁饥饿

## ABA问题



![image-20220320134459672](https://image.imxyu.cn/file/image-20220320134459672.png)



### ABA问题解决AtomicStampedReference

使用版本号时间戳原子引用

```java
package com.atguigu.Interview.study.thread;

import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;

/**
 * @auther zzyy
 * @create 2018-11-20 17:14
 */
public class ABADemo
{
    static AtomicInteger atomicInteger = new AtomicInteger(100);
    static AtomicStampedReference atomicStampedReference = new AtomicStampedReference(100,1);

    public static void main(String[] args)
    {
        new Thread(() -> {
            atomicInteger.compareAndSet(100,101);
            atomicInteger.compareAndSet(101,100);
        },"t1").start();

        new Thread(() -> {
            //暂停一会儿线程
            try { Thread.sleep( 500 ); } catch (InterruptedException e) { e.printStackTrace(); };            System.out.println(atomicInteger.compareAndSet(100, 2019)+"\t"+atomicInteger.get());
        },"t2").start();

        //暂停一会儿线程,main彻底等待上面的ABA出现演示完成。
        try { Thread.sleep( 2000 ); } catch (InterruptedException e) { e.printStackTrace(); }

        System.out.println("============以下是ABA问题的解决=============================");

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t 首次版本号:"+stamp);//1
            //暂停一会儿线程,
            try { Thread.sleep( 1000 ); } catch (InterruptedException e) { e.printStackTrace(); }
            atomicStampedReference.compareAndSet(100,101,atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t 2次版本号:"+atomicStampedReference.getStamp());
            atomicStampedReference.compareAndSet(101,100,atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t 3次版本号:"+atomicStampedReference.getStamp());
        },"t3").start();

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t 首次版本号:"+stamp);//1
            //暂停一会儿线程，获得初始值100和初始版本号1，故意暂停3秒钟让t3线程完成一次ABA操作产生问题
            try { Thread.sleep( 3000 ); } catch (InterruptedException e) { e.printStackTrace(); }
            boolean result = atomicStampedReference.compareAndSet(100,2019,stamp,stamp+1);
            System.out.println(Thread.currentThread().getName()+"\t"+result+"\t"+atomicStampedReference.getReference());
        },"t4").start();
    }
}
 

```