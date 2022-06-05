

# 线程的中断机制



## 什么是中断？

首先，

一个线程不应该由其他线程来强制中断或停止，而是应该由线程自己自行停止。所以，Thread.stop, Thread.suspend, Thread.resume 都已经被废弃了。

其次
在Java中没有办法立即停止一条线程，然而停止线程却显得尤为重要，如取消一个耗时操作。因此，Java提供了一种用于停止线程的机制——中断。

中断只是一种协作机制，Java没有给中断增加任何语法，中断的过程完全需要程序员自己实现。若要中断一个线程，你需要手动调用该线程的interrupt方法，该方法也仅仅是将线程对象的中断标识设成true；
接着你需要自己写代码不断地检测当前线程的标识位，如果为true，表示别的线程要求这条线程中断，
此时究竟该做什么需要你自己写代码实现。

每个线程对象中都有一个标识，用于表示线程是否被中断；该标识位为true表示中断，为false表示未中断；通过调用线程对象的interrupt方法将该线程的标识位设为true；可以在别的线程中调用，也可以在自己的线程中调用。

## 中断的相关API方法



![image-20220319141942537](https://image.imxyu.cn/file/image-20220319141942537.png)



对于下面的几个方法已经不能再用了，因为我们中断线程要通过设置中断标志位，而不是直接stop（）停止（这个相当于Kill -9 ,怎么能直接杀死别的线程呢）

![image-20220319142103166](https://image.imxyu.cn/file/image-20220319142103166.png)

## 面试题：如何使用中断标识停止线程？



### 通过一个volatile变量实现

volatile 就不用多说了，内存可见性。 volatile的变量修改之后，所有线程都会立马感知到

```java
public class InterruptDemo
{
private static volatile boolean isStop = false;

public static void main(String[] args)
{
    new Thread(() -> {
        while(true)
        {
            if(isStop)
            {
                System.out.println(Thread.currentThread().getName()+"线程------isStop = true,自己退出了");
                break;
            }
            System.out.println("-------hello interrupt");
        }
    },"t1").start();

    //暂停几秒钟线程
    try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
    isStop = true;
}

}
```



### 通过AtomicBoolean

Atomic 的类底层使用CAS自旋锁的机制，会不停的去查看，所以这个也是可以的

```java
public class StopThreadDemo
{
    private final static AtomicBoolean atomicBoolean = new AtomicBoolean(true);

    public static void main(String[] args)
    {
        Thread t1 = new Thread(() -> {
            while(atomicBoolean.get())
            {
                try { TimeUnit.MILLISECONDS.sleep(500); } catch (InterruptedException e) { e.printStackTrace(); }
                System.out.println("-----hello");
            }
        }, "t1");
        t1.start();

        try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }

        atomicBoolean.set(false);
    }
}
```



### 通过Thread类自带的中断api方法实现

使用API的方法，通过t1.interrupt()设置标志位为true

```java
public class InterruptDemo
{
    public static void main(String[] args)
    {
        Thread t1 = new Thread(() -> {
            while(true)
            {
                if(Thread.currentThread().isInterrupted())
                {
                    System.out.println("-----t1 线程被中断了，break，程序结束");
                    break;
                }
                System.out.println("-----hello");
            }
        }, "t1");
        t1.start();

        System.out.println("**************"+t1.isInterrupted());
        //暂停5毫秒
        try { TimeUnit.MILLISECONDS.sleep(5); } catch (InterruptedException e) { e.printStackTrace(); }
        t1.interrupt();
        System.out.println("**************"+t1.isInterrupted());
    }
}
 
 
```



#### 实例方法interrupt()【源码】

可以看到上面的注释，很重要

如果当前线程处于join()、wait（）、sleep（） 这三种状态下被打断的话，**会直接抛出异常，并且清空中断标志位(设置为false)**

![image-20220319142721233](https://image.imxyu.cn/file/image-20220319142721233.png)

底层调用的是C++的方法

![image-20220319142704566](https://image.imxyu.cn/file/image-20220319142704566.png)

#### 实例方法isInterrupted

![image-20220319142845978](https://image.imxyu.cn/file/image-20220319142845978.png)



## 中断标志位设置为true不会立刻停止

### 说明

具体来说，当对一个线程，调用 interrupt() 时：

①  如果线程处于正常活动状态，那么会将该线程的中断标志设置为 true，仅此而已。
被设置中断标志的线程将继续正常运行，不受影响。所以， interrupt() 并不能真正的中断线程，需要被调用的线程自己进行配合才行。


②  如果线程处于被阻塞状态（例如处于sleep, wait, join 等状态），在别的线程中调用当前线程对象的interrupt方法，
那么线程将立即退出被阻塞状态，并抛出一个InterruptedException异常。

 

### 代码验证

```java
public class InterruptDemo2
{
    public static void main(String[] args) throws InterruptedException
    {
        Thread t1 = new Thread(() -> {
            for (int i=0;i<300;i++) {
                System.out.println("-------"+i);
            }
            System.out.println("after t1.interrupt()--第2次---: "+Thread.currentThread().isInterrupted());
        },"t1");
        t1.start();

        System.out.println("before t1.interrupt()----: "+t1.isInterrupted());
        //实例方法interrupt()仅仅是设置线程的中断状态位设置为true，不会停止线程
        t1.interrupt();
        //活动状态,t1线程还在执行中
        try { TimeUnit.MILLISECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("after t1.interrupt()--第1次---: "+t1.isInterrupted());
        //非活动状态,t1线程不在执行中，已经结束执行了。
        try { TimeUnit.MILLISECONDS.sleep(3000); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("after t1.interrupt()--第3次---: "+t1.isInterrupted());
    }
}
```



### 在sleep\wait\join 中被中断

当在这几个方法中被中断时，会抛出异常，并且清除标志位设置为false，因此我们必须要在发生interruptedException异常中重新设置标志位为true。 

![image-20220319143057349](https://image.imxyu.cn/file/image-20220319143057349.png)



结论：

![image-20220319143119965](https://image.imxyu.cn/file/image-20220319143119965.png)

### 小总结

中断只是一种协同机制，修改中断标识位仅此而已，不是立刻stop打断



## 静态方法Thread.interrupted()

### 代码演示

```java


/**
 * @auther zzyy
 * @create 2020-05-12 14:19
 * 作用是测试当前线程是否被中断（检查中断标志），返回一个boolean并清除中断状态，
 * 第二次再调用时中断状态已经被清除，将返回一个false。
 */
public class InterruptDemo
{

    public static void main(String[] args) throws InterruptedException
    {
        System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
        System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
        System.out.println("111111");
        Thread.currentThread().interrupt();
        System.out.println("222222");
        System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
        System.out.println(Thread.currentThread().getName()+"---"+Thread.interrupted());
    }
}

>>>>>结果：
    false,
	false,
	true,
	false
       
```

![image-20220319143343329](https://image.imxyu.cn/file/image-20220319143343329.png)



### 静态interrupted和isInterupted

![image-20220319143514060](https://image.imxyu.cn/file/image-20220319143514060.png)

![image-20220319143522104](https://image.imxyu.cn/file/image-20220319143522104.png)



### 结论

 ![image-20220319143540778](https://image.imxyu.cn/file/image-20220319143540778.png)

方法的注释也清晰的表达了“中断状态将会根据传入的ClearInterrupted参数值确定是否重置”。

所以，静态方法interrupted 将会清除中断状态（传入的参数ClearInterrupted为true），

实例方法isInterrupted则不会（传入的参数ClearInterrupted为false）。



## 总结



线程中断相关的方法：

interrupt()方法是一个实例方法
它通知目标线程中断，也就是设置目标线程的中断标志位为true，中断标志位表示当前线程已经被中断了。

isInterrupted()方法也是一个实例方法
它判断当前线程是否被中断（通过检查中断标志位）并获取中断标志

Thread类的静态方法interrupted()
返回当前线程的中断状态(boolean类型)且将当前线程的中断状态设为false，此方法调用之后会清除当前线程的中断标志位的状态（将中断标志置为false了），返回当前值并清零置false



# 线程等待和唤醒机制（引出LockSupport）

## 3种让线程等待和唤醒的方法

### 1、wait/notify

方式1：使用Object中的wait()方法让线程等待，使用Object中的notify()方法唤醒线程

```java
package com.atguigu.juc.prepare;

import java.util.concurrent.TimeUnit;

/**
 * @auther zzyy
 * @create 2020-04-13 17:12
 *
 * 要求：t1线程等待3秒钟，3秒钟后t2线程唤醒t1线程继续工作
 *
 * 1 正常程序演示
 *
 * 以下异常情况：
 * 2 wait方法和notify方法，两个都去掉同步代码块后看运行效果
 *   2.1 异常情况
 *   Exception in thread "t1" java.lang.IllegalMonitorStateException at java.lang.Object.wait(Native Method)
 *   Exception in thread "t2" java.lang.IllegalMonitorStateException at java.lang.Object.notify(Native Method)
 *   2.2 结论
 *   Object类中的wait、notify、notifyAll用于线程等待和唤醒的方法，都必须在synchronized内部执行（必须用到关键字synchronized）。
 *
 * 3 将notify放在wait方法前面
 *   3.1 程序一直无法结束
 *   3.2 结论
 *   先wait后notify、notifyall方法，等待中的线程才会被唤醒，否则无法唤醒
 */
public class LockSupportDemo
{

    public static void main(String[] args)//main方法，主线程一切程序入口
    {
        Object objectLock = new Object(); //同一把锁，类似资源类

        //第一个线程先持有锁，然后进入wait（），此时会等待并且释放锁
        new Thread(() -> {
            synchronized (objectLock) {
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName()+"\t"+"被唤醒了");
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3L); } catch (InterruptedException e) { e.printStackTrace(); }
		
        //第二个线程得到锁进入后唤醒线程1
        new Thread(() -> {
            synchronized (objectLock) {
                objectLock.notify();
            }

            //objectLock.notify();  //1、必须在synchronized里面，否则会抛出异常

            /*synchronized (objectLock) {   //2、必须遵循先wait,后notify 的原则。否则会一直死着
                try {
                    objectLock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }*/
        },"t2").start();



    }
}

 

```



wait /notify 总结：

1、wait和notify方法必须要在同步块或者方法里面，且成对出现使用

否则：

![image-20220319152306368](https://image.imxyu.cn/file/image-20220319152306368.png)

2、先wait后notify才OK，否则程序会被卡死一直在等待

### 2、await/signal

方式2：使用JUC包中Condition的await()方法让线程等待，使用signal()方法唤醒线程

```java
public class LockSupportDemo2
{
    public static void main(String[] args)
    {
        Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();

        new Thread(() -> {
            lock.lock();
            try
            {
                System.out.println(Thread.currentThread().getName()+"\t"+"start");
                condition.await();
                System.out.println(Thread.currentThread().getName()+"\t"+"被唤醒");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            lock.lock();
            try
            {
                condition.signal();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
            System.out.println(Thread.currentThread().getName()+"\t"+"通知了");
        },"t2").start();

    }
}
 
 

```



总结：

1、Condtion中的线程等待和唤醒方法之前，需要先获取锁

2、一定要先await后signal，不要反了

和上面的wait和notify 注意点一样



### 1和2的限制条件

1、线程先要获得并持有锁，必须在锁块(synchronized或lock)中（程序在阻塞中性能很低）

2、必须要先等待后唤醒，线程才能够被唤醒



下面我们来看看LockSupport给我们提供的两个方法

# 3、LockSupport的park/unpark

方式3：LockSupport类可以阻塞当前线程以及唤醒指定被阻塞的线程

## 定义

通过park()和unpark(thread)方法来实现阻塞和唤醒线程的操作

![image-20220319152920810](https://image.imxyu.cn/file/image-20220319152920810.png)


LockSupport是用来创建锁和其他同步类的基本线程阻塞原语。

LockSupport类使用了一种名为Permit（许可）的概念来做到阻塞和唤醒线程的功能， 每个线程都有一个许可(permit)，
permit只有两个值1和零，默认是零。
可以把许可看成是一种(0,1)信号量（Semaphore），但与 Semaphore 不同的是，许可的累加上限是1。



## 主要方法

API：其中我们主要看红色的这俩就可以了

![image-20220319153115358](https://image.imxyu.cn/file/image-20220319153115358.png)



### 阻塞：

![image-20220319153149700](https://image.imxyu.cn/file/image-20220319153149700.png)

阻塞当前线程/阻塞传入的具体线程



### 唤醒：

![image-20220319153226823](https://image.imxyu.cn/file/image-20220319153226823.png)

唤醒处于阻塞状态的指定线程

## 代码

### 正常代码

LockSupport 可以直接使用，不需要锁块、

下面是正常情况： 先让t1暂停，然后让 t2 去唤醒t1 

```java
public class LockSupportDemo3
{
    public static void main(String[] args)
    {
        //正常使用+不需要锁块
Thread t1 = new Thread(() -> {
    System.out.println(Thread.currentThread().getName()+" "+"1111111111111");
    LockSupport.park();
    System.out.println(Thread.currentThread().getName()+" "+"2222222222222------end被唤醒");
},"t1");
t1.start();

//暂停几秒钟线程
try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }

LockSupport.unpark(t1);
System.out.println(Thread.currentThread().getName()+"   -----LockSupport.unparrk() invoked over");

    }
}
 
 
```



### 先unpark再park

如果我们像之前synchronized 和lock 出现异常的第二种情况，先解unpark ,再park呢？

下面我们演示就是：

```java
public class T1
{
    public static void main(String[] args)
    {
        Thread t1 = new Thread(() -> {
            //先等t2 unpark 完了这里再park
            try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(Thread.currentThread().getName()+"\t"+System.currentTimeMillis());
            LockSupport.park();
            System.out.println(Thread.currentThread().getName()+"\t"+System.currentTimeMillis()+"---被叫醒");
        },"t1");
        t1.start();

        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

        LockSupport.unpark(t1);
        System.out.println(Thread.currentThread().getName()+"\t"+System.currentTimeMillis()+"---unpark over");
    }
}
 

```

![image-20220319154447935](https://image.imxyu.cn/file/image-20220319154447935.png)

可以看到并没有任何关系，先执行的t2 相当于把t1中的park（）给提前注释了



### unpark  2(N)次,park 2次



```java
public class Main {

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" );
            LockSupport.park();
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + "\t"  + "---被叫醒");
        }, "t1");
        t1.start();

        new Thread(() -> {
            LockSupport.unpark(t1);
            LockSupport.unpark(t1);
            System.out.println(Thread.currentThread().getName() + "\t"  + "---发出通知");
        }, "t2").start();
    }

}

```

![image-20220319155948039](https://image.imxyu.cn/file/image-20220319155948039.png)

可以看到出现问题了，根据上面api中的解释，unpark 多次只能设置为1，**并且只对一个park 有效。**

也就是是说，

![image-20220319160300265](https://image.imxyu.cn/file/image-20220319160300265.png)



一句话总结： 一个线程只能发一个unpark ，发多了没用. 。也就是一个锅只能对应一个锅盖， t2只能给t1 发一个通行证，发多了没用。上限为1, 只能操作那一个锅



### 如何阻塞两（多）次？

如果我们想让程序阻塞两次怎么办？

那么就需要两个线程去发通行证

并且中间需要一点间隔的发，不能一起发。会出现问题

![image-20220319161211245](https://image.imxyu.cn/file/image-20220319161211245.png)

以此类推，如果我们想阻塞多次，就用多个线程去发。





## 总结

LockSupport是**非阻塞的**，**无锁化**的通知机制，在效率上会有提升。 并且只有两个方法非常好用。

![image-20220319161818206](https://image.imxyu.cn/file/image-20220319161818206.png)

由三点，变成两点。以后都使用LockSupport 就行

底层是Unsafe 类做的，后续会讲



## park后唤醒的两种方式

1、调用unpark方法

2、调用park线程的interrupt 方法

> 这里注意，如果线程死了以后，isInterrupt 就为false了，因为线程已经不处于活跃状态了

![image-20220319184638321](https://image.imxyu.cn/file/image-20220319184638321.png)