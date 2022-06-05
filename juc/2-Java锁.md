



# 1、乐观锁和悲观锁



## 悲观锁

### 定义

认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。

synchronized关键字和Lock的实现类都是悲观锁



### 场景

适合写操作多的场景，先加锁可以保证写操作时数据正确。显式的锁定之后再操作同步资源



只要加锁的，比如下面这俩都是悲观锁

```java
public synchronized void m1()
{
    //加锁后的业务逻辑......
}

// 保证多个线程使用的是同一个lock对象的前提下
ReentrantLock lock = new ReentrantLock();
public void m2() {
    lock.lock();
    try {
        // 操作同步资源
    }finally {
        lock.unlock();
    }
}

```



## 乐观锁

### 定义

乐观锁认为自己在使用数据时**不会有别的线程修改数据**，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。

如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作



乐观锁在Java中是通过使用无锁编程来实现，最常采用的是**CAS**算法，Java原子类中的递增操作就通过CAS自旋实现的。



### 场景

适合读操作多的场景，不加锁的特点能够使其读操作的性能大幅提升。

乐观锁则直接去操作同步资源，是一种无锁算法，得之我幸不得我命，再抢

乐观锁一般有两种实现方式：

1、采用版本号控制

比如解决ABA问题（具体可以看面试必问第二节CAS-ABA中的解决），通过atomicStampedReference 每次加一个版本号

2、CAS （Compare and Swap ，比较并替换） 算法

```java
// 保证多个线程使用的是同一个AtomicInteger
private AtomicInteger atomicInteger = new AtomicInteger();
atomicInteger.incrementAndGet();
```



# synchronized



## 1、synchronized锁的8个案例

这里就不贴代码了，对于普通方法的synchronized 锁的是当前对象，对于static 方法的锁的是当前类对象

遵循能用对象锁就不用类锁，能用代码块就不要加到方法上

## 2、从字节码角度分析synchronized实现

javap -c ***.class文件反编译

假如你需要更多信息

javap -v ***.class文件反编译，输出附加信息（包括行号、本地变量表，反汇编等详细信息）

### 同步代码块

反编译，实现使用的是monitorenter和monitorexit指令

![image-20220319102522324](https://image.imxyu.cn/file/image-20220319102522324.png)



发现有1个enter, 两个exit ？原因是第二个exit 是如果出现异常也要退出保证锁的退出，想当与finally

![image-20220319102535137](https://image.imxyu.cn/file/image-20220319102535137.png)

一定是一个enter两个exit吗？

发现抛出异常的时候首先进入锁enter -》  athrow 抛出异常  -》 锁退出exit -》接住异常

也就是有一个enter, 一个exit ,两个athrow

![image-20220319102710280](https://image.imxyu.cn/file/image-20220319102710280.png)

### 普通同步方法

方法上对应ACC_SYNCHRONIZED 标志位

![image-20220319103014841](https://image.imxyu.cn/file/image-20220319103014841.png)



调用指令将会检查方法的ACC_SYNCHRONIZED访问标志是否被设置。如果设置了，执行线程会将先持有monitor然后再执行方法，
最后在方法完成(无论是正常完成还是非正常完成)时释放 monitor

### 静态同步方法

比普通方法多一个ACC_STATIC 标志，标志为静态方法

![image-20220319103056232](https://image.imxyu.cn/file/image-20220319103056232.png)

ACC_STATIC, ACC_SYNCHRONIZED访问标志区分该方法是否静态同步方法

### 反编译synchronized锁的是什么

#### 管程（monitor）

![image-20220319104103387](https://image.imxyu.cn/file/image-20220319104103387.png)

#### 为什么任何一个对象都可以成为一个锁

所有对象都继承了Object类，在HotSpot虚拟机中，monitor采用ObjectMonitor实现

ObjectMonitor.java→ObjectMonitor.cpp→objectMonitor.hpp

在C++文件中的定义： 对每一个对象都有相应的记录

![image-20220319104234485](https://image.imxyu.cn/file/image-20220319104234485.png)

对于synchronized关键字，我们在《Synchronized与锁升级》章节还会再深度讲解

synchronized必须作用于某个对象中，所以Java在对象的头文件存储了锁的相关信息。锁升级功能主要依赖于 MarkWord 中的锁标志位和释放偏向锁标志位，后续讲解锁升级时候我们再加深，目前为了承前启后的学习，对下图先混个眼熟即可

包括我们的hashcode, 还有锁的标志位的信息，都会记录在对象的头文件中

![image-20220319104512328](https://image.imxyu.cn/file/image-20220319104512328.png)



# 3、公平锁和非公平锁



**公平锁**：就是很公平，在并发环境中，每个线程在获取锁时会先查看此锁维护的等待队列，如果为空，或者当前线程是等待队列中的第一个，就占用锁，否者就会加入到等待队列中，以后安装FIFO的规则从队列中取到自己

**非公平锁：** 非公平锁比较粗鲁，上来就直接尝试占有锁，如果尝试失败，就再采用类似公平锁那种方式。

## 卖票问题演示

```java
package com.atguigu.juc.senior.test;

import java.util.concurrent.locks.ReentrantLock;

class Ticket
{
    private int number = 30;
    //默认是创建的非公平锁，参数默认是false
     Lock lock = new ReentrantLock();


    public void sale()
    {
        lock.lock();
        try
        {
            if(number > 0)
            {
                System.out.println(Thread.currentThread().getName()+"卖出第：\t"+(number--)+"\t 还剩下:"+number);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}


/**
 * @auther zzyy
 * @create 2020-05-14 17:26
 */
public class SaleTicketDemo
{
    public static void main(String[] args)
    {
        Ticket ticket = new Ticket();

        new Thread(() -> { for (int i = 0; i <35; i++)  ticket.sale(); },"a").start();
        new Thread(() -> { for (int i = 0; i <35; i++)  ticket.sale(); },"b").start();
        new Thread(() -> { for (int i = 0; i <35; i++)  ticket.sale(); },"c").start();
    }
}
 
 

```

结果： 全让a给抢到了， b和c 饿死

![image-20220319105932472](https://image.imxyu.cn/file/image-20220319105932472.png)

当我们修改为公平锁：

![image-20220319110015021](https://image.imxyu.cn/file/image-20220319110015021.png)

结果：

![image-20220319110036241](https://image.imxyu.cn/file/image-20220319110036241.png)

## 底层源码

可以看到根据参数的不同创建的是公平锁还是非公平锁，默认是创建的非公平锁

![image-20220319110206755](https://image.imxyu.cn/file/image-20220319110206755.png)

按序排队公平锁，就是判断同步队列是否还有先驱节点的存在(我前面还有人吗?)，如果没有先驱节点才能获取锁；
先占先得非公平锁，是不管这个事的，只要能抢获到同步状态就可以

![image-20220319110236138](https://image.imxyu.cn/file/image-20220319110236138.png)



## 为什么会有公平锁/非公平锁的设计为什么默认非公平？

1、恢复挂起的线程到真正锁的获取还是有时间差的，从开发人员来看这个时间微乎其微，但是从CPU的角度来看，这个时间差存在的还是很明显的。所以非公平锁能更充分的利用CPU 的时间片，尽量减少 CPU 空闲状态时间。

2、使用多线程很重要的考量点是线程切换的开销，当采用非公平锁时，当1个线程请求锁获取同步状态，然后释放同步状态，因为不需要考虑是否还有前驱节点，所以刚释放锁的线程在此刻再次获取同步状态的概率就变得非常大，所以就减少了线程的开销。

## 公平锁造成的问题

公平锁保证了排队的公平性，非公平锁霸气的忽视这个规则，所以就有可能导致排队的长时间在排队，也没有机会获取到锁，这就是传说中的 “锁饥饿”



## 何时用公平锁，何时用非公平


如果为了更高的吞吐量，很显然非公平锁是比较合适的，因为节省很多线程切换时间，吞吐量自然就上去了；否则那就用公平锁，大家公平使用。



## 预埋伏AQS

底层的公平锁和非公平锁都继承了Sync ,而Sync 继承了Aqs 。更进一步的源码深度分析见后续第13章

![image-20220319110616200](https://image.imxyu.cn/file/image-20220319110616200.png)



![image-20220319110637849](https://image.imxyu.cn/file/image-20220319110637849.png)



# 4、可重入锁（递归锁）

可重入锁又名递归锁

是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁(前提，锁对象得是同一个对象)，不会因为之前已经获取过还没释放而阻塞。

如果是1个有 synchronized 修饰的递归调用方法，程序第2次进入被自己阻塞了岂不是天大的笑话，出现了作茧自缚。
所以Java中ReentrantLock和synchronized都是可重入锁，可重入锁的一个优点是可一定程度避免死锁。

## 隐式锁（synchronized）默认是可重入锁

同步代码块：

```java
public class ReEntryLockDemo
{
    public static void main(String[] args)
    {
        final Object objectLockA = new Object();

        new Thread(() -> {
            synchronized (objectLockA)
            {
                System.out.println("-----外层调用");
                synchronized (objectLockA)
                {
                    System.out.println("-----中层调用");
                    synchronized (objectLockA)
                    {
                        System.out.println("-----内层调用");
                    }
                }
            }
        },"a").start();
    }
}
 
```

同步方法：

```java
public class ReEntryLockDemo
{
    public synchronized void m1()
    {
        System.out.println("-----m1");
        m1();
    }
  

    public static void main(String[] args)
    {
        ReEntryLockDemo reEntryLockDemo = new ReEntryLockDemo();

        reEntryLockDemo.m1();
    }
}
 

```

> 可以看到都是可以进入的，不会因为自己持有了锁，而调用使用 这个锁的方法或者同步代码块而有限制，这没什么好说的吧



## 可重入锁原理

之前我们说过每个锁对象拥有一个锁计数器和一个指向持有该锁的线程的指针。

![image-20220319115429708](https://image.imxyu.cn/file/image-20220319115429708.png)

当执行monitorenter时，如果目标锁对象的计数器（_count）为零，那么说明它没有**被其他线程所持有**，Java虚拟机会将该锁对象的持有线程设置为**当前线程**，并且将其计数器加1。

在目标锁对象的计数器不为零的情况下，如果锁对象的持有线程是**当前线程，那么 Java 虚拟机可以将其计数器加1**，否则需要等待，直至持有线程释放该锁。

当执行monitorexit时，Java虚拟机则需将锁对象的计数器减1。计数器为零代表锁已被释放。

## 显式锁（即Lock）也有ReentrantLock这样的可重入锁。

```java
public class ReEntryLockDemo
{
    static Lock lock = new ReentrantLock();

    public static void main(String[] args)
    {
        new Thread(() -> {
            lock.lock();
            try
            {
                System.out.println("----外层调用lock");
                lock.lock();
                try
                {
                    System.out.println("----内层调用lock");
                }finally {
                    // 这里故意注释，实现加锁次数和释放次数不一样
                    // 由于加锁次数和释放次数不一样，第二个线程始终无法获取到锁，导致一直在等待。
                    lock.unlock(); // 正常情况，加锁几次就要解锁几次
                }
            }finally {
                lock.unlock();
            }
        },"a").start();

        new Thread(() -> {
            lock.lock();
            try
            {
                System.out.println("b thread----外层调用lock");
            }finally {
                lock.unlock();
            }
        },"b").start();

    }
}
 
```

唯一一个要注意的一点是，lock 加锁几次就要解锁几次，否则其他线程会永远得不到锁，会一直卡住



# 5、死锁

![image-20220319120253527](https://image.imxyu.cn/file/image-20220319120253527.png)

 

死锁产生的主要原因：

* 系统资源不足
* 进程运行推进的顺序不当
* 资源分配不当

## 手写一个死锁代码（重点）



```java
public class DeadLockDemo
{
    public static void main(String[] args)
    {
        final Object objectLockA = new Object();
        final Object objectLockB = new Object();

        new Thread(() -> {
            synchronized (objectLockA)
            {
                System.out.println(Thread.currentThread().getName()+"\t"+"自己持有A，希望获得B");
                //暂停几秒钟线程
                try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
                synchronized (objectLockB)
                {
                    System.out.println(Thread.currentThread().getName()+"\t"+"A-------已经获得B");
                }
            }
        },"A").start();

        new Thread(() -> {
            synchronized (objectLockB)
            {
                System.out.println(Thread.currentThread().getName()+"\t"+"自己持有B，希望获得A");
                //暂停几秒钟线程
                try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
                synchronized (objectLockA)
                {
                    System.out.println(Thread.currentThread().getName()+"\t"+"B-------已经获得A");
                }
            }
        },"B").start();

    }
}
 
```

![image-20220319123041170](https://image.imxyu.cn/file/image-20220319123041170.png)

## 如何证明这是一个死锁？（死锁排查）

并不是说程序一直在运行就能说明有死锁，可能是死循环，或者服务调用之间的等待。

需要使用下面的命令去查看



### **1、使用Jconsole**

cmd 输入jconsole

![image-20220319122739262](https://image.imxyu.cn/file/image-20220319122739262.png)



![image-20220319122818358](https://image.imxyu.cn/file/image-20220319122818358.png)

![image-20220319122824902](https://image.imxyu.cn/file/image-20220319122824902.png)

![image-20220319122829433](https://image.imxyu.cn/file/image-20220319122829433.png)

### **2、使用jps -l +jstack 进程编号**

jps 命令相当于linux中的ps -ef ,	jps这个是专门查看java进程的

![image-20220319122948701](https://image.imxyu.cn/file/image-20220319122948701.png)

![image-20220319123015010](https://image.imxyu.cn/file/image-20220319123015010.png)



6、其他后面补充（等）

![image-20220319123148748](https://image.imxyu.cn/file/image-20220319123148748.png)



### 小问题（看不到java 进程）

通过jps 或者jconsole ，如果看不到所有的java 进程号，可能是用户权限问题



解决办法： 在对应的用户文件夹开启用户权限即可： 

https://cloud.tencent.com/developer/article/1039894