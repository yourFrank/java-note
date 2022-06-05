---
title: 0-Synchronized详解
tags:
  - 多线程基础
categories:
  - 多线程基础
description: java重要关键字——Synchronized
cover: 'https://image.imxyu.cn/file/synchroni.png'
abbrlink: 311ad3
date: 2021-12-14 22:21:58
---

## Synchronized简介

官方的解释很晦涩，一句话解释该关键字的作用：

能够保证在同一时刻最多只有一个线程执行该段代码，以达到保证并发安全的效果。 

> 具体通过一把锁来控制，第一个线程会持有该锁，拥有该锁的线程才能执行这段代码，只有该锁被释放了其他线程获取到该锁才能执行，后续会详讲

Synchronized是地位很重要：

* Synchronized是java中的关键字，被java原生支持
* 是最基本的互斥同步手段
* 并发编程必学内容！



## 不使用并发的后果

代码实战： 两个线程同时i++,最后结果会比预期的少

```java
public class ShowUnsafe1 implements Runnable{

    static ShowUnsafe1 r=new ShowUnsafe1();
    static int i=0;
    @Override
    public void run() {
        for (int j = 0; j < 100000; j++) {
            i+=1;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(r);
        Thread t2=new Thread(r);
        t1.start();
        t2.start();
        t1.join();//join()可以让t1方法执行完毕后再运行下面的,也就是main线程要等t1执行完再执行下面的
        t2.join();
        System.out.println(i);

    }
}
```

结果：

![image-20211214201542237](https://image.imxyu.cn/file/image-20211214201542237.png)

我们可能会想，结果应该是20W，但是没有我们预期的那样，而且每次结果都不一致

原因：

i++虽然是一行代码，但是实际包含了三个动作:

1. 读取i的值
2. 计算i+1
3. 把i+1的计算结果写到内存中，赋给i

这三步并不是原子操作，这就会出现线程1读取完i ，执行i +1 此时还没来得及将i+1写回到内存中，线程2

此时来读取i，此时还是10，因为线程1还没写回去。这样两个线程就只将i从10加到了11，我们是想让他加到12。原本是想加20W次，结果很多次都丢失了！

> 原子操作就是这三项整体是不可分割的 

## Synchronized两种用法

### 对象锁

包括**方法(普通方法)锁**（默认锁对象为this当前实例对象）和**同步代码块锁**（自己指定锁对象）

#### 同步代码块

我们来模拟一段代码

```java
public class SynchronizedObjectCodeBlock2 implements Runnable {
    private static SynchronizedObjectCodeBlock2 instant = new SynchronizedObjectCodeBlock2();

    @Override
    public void run() {
        System.out.println("我是对象锁的代码块形式，我叫"+Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName()+" 运行结束");

    }


    public static void main(String[] args) {
        Thread t1=new Thread(instant);
        Thread t2=new Thread(instant);
        t1.start();
        t2.start();
//        t1.join();
//        t2.join();
        while (t1.isAlive()||t2.isAlive()){//换一种方式这里如果存活一直循环，也就是等t1或者t2执行完再执行后面的
        }
        System.out.println("finished ");
    }
}
```

看结果此时两个线程是同时执行，同时结束的

![image-20211214204149511](https://image.imxyu.cn/file/image-20211214204149511.png)



如果我们想让他们有一个先后顺序，可以用Synchronized关键字：

![image-20211214204316588](https://image.imxyu.cn/file/image-20211214204316588.png)



此时我们用的锁时是this代表当前对象，我们也可以自己指定锁对象

![image-20211214204548144](https://image.imxyu.cn/file/image-20211214204548144.png)

执行结果和刚才的this一样，那么我们为什么要自己指定锁对象呢？

原因是在业务复杂的情况下，可能会在一个方法内部出现多把锁的情况，并且操作的这两段代码互不干扰 

![image-20211214205604740](https://image.imxyu.cn/file/image-20211214205604740.png)

对于第二个线程要等t1释放lock1后才能获取lock1执行，然后运行完lock1中代码后并且t1释放lock2后执行第二段代码

#### debug查看线程技巧

![image-20211214210238335](https://image.imxyu.cn/file/image-20211214210238335.png)



此时我们可以看到哪个线程在运行，哪个在等锁

![image-20211214210458265](https://image.imxyu.cn/file/image-20211214210458265.png)



![image-20211214210619818](https://image.imxyu.cn/file/image-20211214210619818.png)

#### 同步普通方法

![image-20211214211202403](https://image.imxyu.cn/file/image-20211214211202403.png)

效果也是和代码块一样的

> 我们再来看一下之前说的：

对于**方法(普通方法)锁**我们没有指定锁对象，因此默认是this 当前对象，对于**同步代码块**锁（自己指定锁对象）

### 类锁

指Synchronized 修饰静态的方法或指定锁为Class对象

#### 重要概念

**Java类可能有很多对象，但只有1个Class对象**

 对于一个类的实例可能有很多个，但是类对象只有一个

**本质**：所谓的类锁，不过是Class对象的锁而已

**用法和效果**：类锁只能在同一时刻被一个对象拥有。之前的**对象锁**如果**不同的对象**互相不会影响，都可以运行。但是对于类锁只能一个运行（因为只有一个类对象）

#### synchronized 加到static方法

之前我们讲synchronized 修饰**普通方法**的时候，锁对象默认为this

我们看一下刚才的实例，如果使用两个对象的时候

![image-20211214212720846](https://image.imxyu.cn/file/image-20211214212720846.png)

因为普通方法，锁默认为this(当前对象),这里使用了两个对象，因此锁是不一样的，互不干扰。因此是同时运行，同时结束

此时如果该方法设置为**static**, 然后设置为**synchronized**  该方法就升级为类锁了，就又可以等待执行了

![image-20211214214205781](https://image.imxyu.cn/file/image-20211214214205781.png)



#### synchronized （*.class）代码块

```java
public class SynchronizedClass5 implements Runnable {
    private static SynchronizedClass5 instant1 = new SynchronizedClass5();
    private static SynchronizedClass5 instant2 = new SynchronizedClass5();

    @Override
    public void run() {
        method();
    }

    private void method() {
        synchronized (this) {
            System.out.println("我是对象锁的方法形式，我叫" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);//休眠3秒
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " 运行结束");
        }

    }

    //下面和之前一样创建两个线程
    public static void main(String[] args) {
        Thread t1 = new Thread(instant1);
        Thread t2 = new Thread(instant2);
        t1.start();
        t2.start();
        while (t1.isAlive() || t2.isAlive()) {//换一种方式这里如果存活一直循环，也就是等t1或者t2执行完再执行后面的
        }
        System.out.println("finished ");
    }

}
```

如果我们使用**synchronized (this){ }**代码块的形式，使用this锁同上，对于两个对象锁不一样，因此还是会同时执行

当我们使用synchronized （.class）变成类锁，类锁只有一个，因此还是会顺序执行。

> 注意这里的类可以是其他任意类，不一定是表示线程对象的这个类。因为任意**类对象**都是唯一的，都可以。

![image-20211214213613846](https://image.imxyu.cn/file/image-20211214213613846.png)

## 使用并发解决之前i++

我们学会了类锁和对象锁，我们就可以解决刚才两个线程i++的问题了

1. 使用对象锁同步方法

![image-20211214214723103](https://image.imxyu.cn/file/image-20211214214723103.png)

2. 使用对象锁同步代码块

   ```java
   public void run() {
       synchronized (this){
           for (int j = 0; j < 100000; j++) {
               i+=1;
           }
       }
   }
   ```

3. 使用类锁

   ```java
   @Override
   public void run() {
       synchronized (ShowSafe.class){
           for (int j = 0; j < 100000; j++) {
               i+=1;
           }
       }
   }
   ```

我们都可以达到最终的效果

## 多线程访问同步方法的7种情况

1. 两个线程同时访问一个对象的同步方法

   上面说过这种是可以实现保护的，使用的同一个锁

2. 两个线程访问的是两个对象的同步方法

   锁不同，不能保护

3. 两个线程访问的是synchronized的静态方法

    使用的是类锁，可以保护

4. (同时)一个线程访问同步方法与另一个线程访问非同步方法

   同时运行，两者互不影响

5. 访问同一个对象的不同的普通同步方法

   ```java
   /**
    * 访问同一个对象的不同的普通同步方法
    */
   public class SynchronizedDifferentMethod7 implements Runnable {
       private static SynchronizedDifferentMethod7 instance = new SynchronizedDifferentMethod7();
   
       @Override
       public void run() {
           if (Thread.currentThread().getName().equals("Thread-0")){ //如果是Thread0运行method1
               method1();
           }else{ //如果是Thread1运行method2
               method2();
           }
       }
   
       private  synchronized void method1() {
               System.out.println("我是加锁的方法1，我叫" + Thread.currentThread().getName());
               try {
                   Thread.sleep(3000);//休眠3秒
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               System.out.println(Thread.currentThread().getName() + " 运行结束");
   
       }
       private  synchronized void method2() {
           System.out.println("我是加锁的方法2，我叫" + Thread.currentThread().getName());
           try {
               Thread.sleep(3000);//休眠3秒
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           System.out.println(Thread.currentThread().getName() + " 运行结束");
   
       }
   
   
       //下面和之前一样创建两个线程
       public static void main(String[] args) {
           Thread t1 = new Thread(instance);
           Thread t2 = new Thread(instance);
           t1.start();
           t2.start();
           while (t1.isAlive() || t2.isAlive()) {//换一种方式这里如果存活一直循环，也就是等t1或者t2执行完再执行后面的
           }
           System.out.println("finished ");
       }
   
   }
   ```

   得到的结果是按照顺序执行的，因为两个方法的锁都是对象的锁，而二者都是同一个对象因此使用同一个锁，t1拿到后t2要等待释放后才能执行

6. 同时访问静态synchronized和非静态synchronized方法

   同时运行的，因为static类锁，非静态是this对象锁。不同的锁二者互不干扰

7. 方法抛出异常后释放锁

将上面5的method1添加throw new RuntimeException() 后：

![image-20211215184142522](https://image.imxyu.cn/file/image-20211215184142522.png)



可以看到当一个线程抛出异常后，这个锁会被释放，另一个线程就会拿到锁执行

## Synchronized的性质

1. 可重入：指的是同一线程的外层函数获得锁之后，不需要释放锁内层函数可以直接再次获取该锁

> 不可重入的意思就是外层函数获得锁之后，内层函数要获取锁要等该外层函数释放后和其他线程竞争获取。
>
> 好处：避免死锁、提升封装性

```java
/**
 * 可重入举例1： 递归调用本方法
 */
public class SynchronizedRecursion10 {
    int a=0;

    public synchronized void method1(){
        System.out.println("这是method1,a= "+a);
        if (a==0){
            a++;
            method1();//此处可以直接调用，说明是可重入的
        }
    }

    public static void main(String[] args) {
        SynchronizedRecursion10 s=new SynchronizedRecursion10();
        s.method1();
    }
}

```

```java
/**
 * 可重入举例2：调用类内的另外方法
 */
public class SynchronizedIOtherMethod11 {
    public synchronized void method1(){
        System.out.println("这是method1");
        method2();
    }
    //本来进入方法2之前要拿到这个锁的，但是已经method1已经持有了该锁可以直接调用另一个synchronized方法，不需要释放的动作可以直接执行。可重入
    private synchronized void method2() {
        System.out.println("这是method2");
    }

    public static void main(String[] args) {
        SynchronizedIOtherMethod11 s=new SynchronizedIOtherMethod11();
        s.method1();
    }
}

```



2. 不可中断: 一旦这个锁已经被别人获得了，如果我还想获得，我只能选择等待或者阻塞，直到别的线程释放了这个锁。如果别人永远不释放锁，那么我只能永远等下去。

## Synchronized原理

### 加锁和释放锁的原理

对于Synchronized获取和释放锁的时机：进入和退出同步代码块（包括抛出异常）

#### Synchronized等价的代码Lock锁

等价代码：

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Synchronized加锁和释放锁的等价代码，Lock的形式表现
 */
public class SynchronizedToLock13 {

    Lock lock = new ReentrantLock();
    //对于synchronized，进入synchronized方法就是获得了锁，退出就相当于释放了锁。和下面那种形式是等价的，内部帮我们做了这些
    public synchronized void method1() {
        System.out.println("我是synchronized形式");
    }

    //lock锁，和synchronized关键字等价的代码
    private void method2() {
        //上锁
        lock.lock();
        //这里一定要用try,finally，否则try中出现了异常可能会导致锁没被释放出现问题
        try {
            System.out.println("我是Lock的形式");
        } finally {
            lock.unlock(); //释放锁
        }
    }

    public static void main(String[] args) {
        SynchronizedToLock13 s = new SynchronizedToLock13();
        s.method1();
        s.method2();
    }
}
```

#### 对应字节码:monitor指令

1. 首先我们创建一个方法，我们来看一下底层字节码是如何实现锁的

```java
/**
 * 看字节码
 */
public class Decompilation14 {
    private Object object=new Object();

    public void insert(Thread thread){
        synchronized (object){

        }
    }
}

```

2. 找到相应的文件，javac编译成class

![image-20211215195309331](https://image.imxyu.cn/file/image-20211215195309331.png)

![image-20211215195513998](C:\Users\tianyu\AppData\Roaming\Typora\typora-user-images\image-20211215195513998.png)

2. javap -verbose 将class文件信息打印出来

![image-20211215195650934](https://image.imxyu.cn/file/image-20211215195650934.png)

也就是说我们写了synchronized (object){}同步代码块或者方法，java会转换成monitor相关指令，执行到相应的代码就会执行monitorenter如果拿到锁就执行，没有就等待

背后的原理就是monitorenter和monitorexit指令

### 可重入性原理

![image-20211215200026455](https://image.imxyu.cn/file/image-20211215200026455.png)

### 可见性

![image-20211215200152208](https://image.imxyu.cn/file/image-20211215200152208.png)

也就是说如果两个方法都加了Synchronized字段，可见性是可以保证的，第一个线程执行的第一个方法的结果第二个线程在第二个方法里是可以得到第一个线程在第一个方法里的执行结果 。如果这两个方法不加Synchronized。可能第一个线程在第一个方法中对x操作后，第二个线程在第二个方法获得的x是不一定的，因为没加Synchronized保护

## Synchronized的缺陷

* 效率低∶锁的释放情况少、试图获得锁时不能设定超时、不能中断一个正在试图获得锁的线程（比如说读io或者sleep等阻塞的方法耗时的操作会等很长时间，其他线程只能等待他释放锁非常影响程序执行效率）

>  Lock锁就可以解决上面的问题

* 效率低∶锁的释放情况少、试图获得锁时不能设定超时、不能中断一个正在试图获得锁的线程
  不够灵活（读写锁更灵活)︰加锁和释放的时机单一，每个锁仅有单一的条件(某个对象），可能是不够的（有一种读写锁， 在读的时候是不会锁住的，因为读不会有问题，只有写的时候需要锁，这种是非常灵活的）
* 无法知道是否成功获取到锁

## 常见面试问题

1. 使用注意点:锁的范围不宜过大、避免锁的嵌套

2. 如何选择Lock和synchronized关键字?

   我们前面说了Lock锁很灵活，并且可以设定超时时间等。但是如果写的不规范，没有写try,finally或者忘记释放锁就会出现问题。所以如果能使用synchronized解决的，就优先使用synchronized。前面说了因为synchronized进入和退出底层帮我们做了加锁和释放锁的过程，当然如果有特殊需要的话使用Lock锁

3. 多线程访问同步方法的各种具体情况

   前面讲的7种情况，只要注意锁的对象就行

4. Synchronized使得同时只有一个线程可以执行，性能较差，有什么办法可以提升性能?

   可以使用读写锁，在读的时候可以有多个线程同时执行

5. 我想更灵活地控制锁的获取和释放（现在释放锁的时机都被规定死了），怎么办?

   synchronized时机很固定就是进入代码块和出代码块，如果我们想灵活控制可以使用Lock

> 一句话介绍synchronized
> JVM会自动通过使用monitor来加锁和解锁，保证了同时只有一个线程可以执行指定代码，从而保证了线程安全，同时具有可重入和不可中断的性质。
