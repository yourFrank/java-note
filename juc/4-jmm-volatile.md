# JMM内存模型

## 计算机硬件存储及体系

上面寄存器是我们的cpu， cpu的速度很快，比主内存快。因此要经过一层缓存，而这个缓存和主内存之间的一致性协议就是我们jmm定义的规范

![image-20220319165518050](https://image.imxyu.cn/file/image-20220319165518050.png)

因为有这么多级的缓存(cpu和物理主内存的速度不一致的)，
CPU的运行并不是直接操作内存而是先把内存里边的数据读到缓存，而内存的读和写操作的时候就会造成不一致的问题

![image-20220319165652712](https://image.imxyu.cn/file/image-20220319165652712.png)

Java虚拟机规范中试图定义一种Java内存模型（java Memory Model，简称JMM) 来屏蔽掉各种硬件和操作系统的内存访问差异，
以实现让Java程序在各种平台下都能达到一致的内存访问效果。推导出我们需要知道JMM



## 定义


JMM(Java内存模型Java Memory Model，简称JMM)本身是一种抽象的概念并不真实存在它仅仅描述的是一组约定或规范**，通过这组规范定义了程序中(尤其是多线程)各个变量的读写访问方式并决定一个线程对共享变量的写入何时以及如何变成对另一个线程可见，关键技术点都是围绕多线程的原子性、可见性和有序性展开的。**

 原则：
 JMM的关键技术点都是围绕多线程的原子性、可见性和有序性展开的

**能干嘛？**

1 通过JMM来实现**线程和主内存之间的抽象关系。**
2 **屏蔽各个硬件平台和操作系统的内存访问差异**以实现让Java程序在各种平台下都能达到一致的内存访问效果。

## JMM规范下三大特性

### 可见性

是指当一个线程修改了某一个共享变量的值，其他线程是否能够立即知道该变更 ，JMM规定了所有的变量都存储在主内存中。
![image-20220319170352966](https://image.imxyu.cn/file/image-20220319170352966.png)

Java中普通的共享变量不保证可见性，因为数据修改被写入内存的时机是不确定的，多线程并发下很可能出现"脏读"，所以每个线程都有自己的工作内存，线程自己的工作内存中保存了该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作（读取，赋值等 ）都必需在线程自己的工作内存中进行，而不能够直接读写主内存中的变量。不同线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成



 线程脏读：如果没有可见性保证
![image-20220319170335329](https://image.imxyu.cn/file/image-20220319170335329.png)

 

 

 

###  原子性

指一个操作是不可中断的，即多线程环境下，操作不能被其他线程干扰

### 有序性

![image-20220319170446016](https://image.imxyu.cn/file/image-20220319170446016.png)

> 如果两条语句没有依赖性的话，是可能会被重排的

这些具体看juc 面试那节



## JMM多线程读取过程

![image-20220319175001534](https://image.imxyu.cn/file/image-20220319175001534.png)

## 总结

1、我们定义的所有共享变量都储存在物理主内存中

2、每个线程都有自己独立的工作内存，里面保存该线程使用到的变量的副本(主内存中该变量的一份拷贝)

3、线程对共享变量所有的操作都必须先在线程自己的工作内存中进行后写回主内存，不能直接从主内存中读写(不能越级)

4、不同线程之间也无法直接访问其他线程的工作内存中的变量，线程间变量值的传递需要通过主内存来进行(同级不能相互访问)



## JMM规范下，多线程先行发生原则之happens-before（要会背）

在JMM中，如果一个操作执行的结果需要对另一个操作可见性
或者 代码重排序，那么这两个操作之间必须存在happens-before关系。

![image-20220319181127865](https://image.imxyu.cn/file/image-20220319181127865.png)



在单线程的情况下，我们不需要去关心顺序性就是在JMM的原则下控制的 main 方法从上向下执行

![image-20220319181138487](https://image.imxyu.cn/file/image-20220319181138487.png)

### happens-before 总原则

1、如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。

2、两个操作之间存在happens-before关系，并不意味着一定要按照happens-before原则制定的顺序来执行。
如果重排序之后的执行结果与按照happens-before关系来执行的结果一致，那么这种重排序并不非法。



### happens-before之8条

1、**次序规则**：一个线程内，按照代码顺序，写在前面的操作先行发生于写在后面的操作；

加深说明：前一个操作的结果可以被后续的操作获取。讲白点就是前面一个操作把变量X赋值为1，那后面一个操作肯定能知道X已经变成了1。

2、**锁定规则**：一个unLock操作先行发生于后面((这里的“后面”是指时间上的先后))对同一个锁的lock操作；

```java
public class HappenBeforeDemo
{
    static Object objectLock = new Object();

    public static void main(String[] args) throws InterruptedException
    {
        //对于同一把锁objectLock，threadA一定先unlock同一把锁后B才能获得该锁，   A 先行发生于B
        synchronized (objectLock)
        {

        }
    }
}

```

3、**volatile变量规则**：对一个volatile变量的写操作先行发生于后面对这个变量的读操作，前面的写对后面的读是可见的，这里的“后面”同样是指时间上的先后。

4、**传递规则**：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C；

5、**线程启动规则(**Thread Start Rule)：Thread对象的start()方法先行发生于此线程的每一个动作

6、**线程中断规则**(Thread Interruption Rule)：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生；可以通过Thread.interrupted()检测到是否发生中断

7、**线程终止规则**(Thread Termination Rule)：线程中的所有操作都先行发生于对此线程的终止检
测，我们可以通过Thread::join()方法是否结束、Thread::isAlive()的返回值等手段检测线程是否已经终止执行。

8、**对象终结规则(**Finalizer Rule)：一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始。

说人话:对象没有完成初始化之前，是不能调用finalized()方法的

> 前四条是针对单线程下说的，后四条是针对多线程下说的



### 案例说明

![image-20220319181937184](https://image.imxyu.cn/file/image-20220319181937184.png)

![image-20220319181950251](https://image.imxyu.cn/file/image-20220319181950251.png)





修复：

方法1、把getter/setter方法都定义为synchronized方法

方法2、把value定义为volatile变量，由于setter方法对value的修改不依赖value的原值，满足volatile关键字使用场景



> 总结，happens-before 只是一种规范是一种理论，而我们的synchronized 和volatile 就是对这个理论落地的实现 ，保证了前后的顺序性。如果我们有前后的依赖，使用synchronized 和volatile  就可以解决顺序性的问题，而这两个关键字遵循的就是happens -before 的理念



# volatile

上面讲的jmm内存模型是理念，底层有很多技术来支持，其中一个就是volatile

## 被volatile修改的变量有两大特点：

特点：

1、可见性

2、有序性（禁止重排序）

![image-20220319190148492](https://image.imxyu.cn/file/image-20220319190148492.png)

## 内存屏障（重点必须会）

### 内存屏障是什么？

 ![image-20220319190823128](https://image.imxyu.cn/file/image-20220319190823128.png)

![image-20220319190922756](https://image.imxyu.cn/file/image-20220319190922756.png)



### volatile凭什么可以保证可见性和有序性？？？

内存屏障 (Memory Barriers / Fences)

### 四个类中调用内存屏障指令

上面我们说过happens-before 先行发生原则是规范，而volatile 是落地的实现，那么它到底如何保证的？

从下面这开始找

![image-20220319191601840](https://image.imxyu.cn/file/image-20220319191601840.png)



unsafe.class 这个类是java能操作底层的一个类

![image-20220319191728061](https://image.imxyu.cn/file/image-20220319191728061.png)

对应C++源码unsafe.cpp：

![image-20220319191807337](https://image.imxyu.cn/file/image-20220319191807337.png)

头文件定义：OrderAccess.hpp

![image-20220319191837234](https://image.imxyu.cn/file/image-20220319191837234.png)

对应cpu的四个指令：

![image-20220319191857127](https://image.imxyu.cn/file/image-20220319191857127.png)





### 内存屏障源码分析（重点）

![image-20220319192559363](https://image.imxyu.cn/file/image-20220319192559363.png)



### happens-before 之 volatile 变量规则

普通读写就是没有标volatile的读写

![image-20220319194336580](https://image.imxyu.cn/file/image-20220319194336580.png)

### JMM 就将内存屏障插⼊策略分为 4 种

#### volatile写：

前后插入

volatile写之前插入storestore （保证写的顺序），storeLoad（写之后插入storeload屏障，写完了再读防止读到脏数据）

![image-20220319202545687](https://image.imxyu.cn/file/image-20220319202545687.png)

happens-before 先行发生原则的保障，顺序性

![image-20220319194439414](https://image.imxyu.cn/file/image-20220319194439414.png)



#### voltile读：

在后面插入一个loadload(禁止下面所有的普通读和voltaile读与当前的voltile读重排序，**这条保证了voltile读的时候要从主内存中取最新的**，本地现在读的都无效，因此要先voltile读), 在后面插入要给loadstore (禁止下面的写和当前voltaile读重排序，保证读到的都是这一刻最新的)

![image-20220319194502190](https://image.imxyu.cn/file/image-20220319194502190.png)



对应这张图想一下，这么做就是为了防止得到脏数据

![image-20220319190148492](https://image.imxyu.cn/file/image-20220319190148492.png)

#### 总结

![image-20220319194410312](https://image.imxyu.cn/file/image-20220319194410312.png)

## volatile特性

### 保证可见性

**保证不同线程对这个变量进行操作时的可见性，即变量一旦改变所有线程立即可见**

```java
public class VolatileSeeDemo
{
    static          boolean flag = true;       //不加volatile，没有可见性
    //static volatile boolean flag = true;       //加了volatile，保证可见性

    public static void main(String[] args)
    {
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"\t come in");
            while (flag)
            {

            }
            System.out.println(Thread.currentThread().getName()+"\t flag被修改为false,退出.....");
        },"t1").start();

        //暂停2秒钟后让main线程修改flag值
        try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }

        flag = false;

        System.out.println("main线程修改完成");
    }
}

```

不加volatile，没有可见性，程序无法停止

加了volatile，保证可见性，程序可以停止



![image-20220319203539410](https://image.imxyu.cn/file/image-20220319203539410.png)

### volatile变量的读写过程

![image-20220319203647406](https://image.imxyu.cn/file/image-20220319203647406.png)

read->load->use->assign->store  这些不用解释了，当我们从write的时候，为了保证同一时刻只能有一个写，底层引入了lock,unlock(这个不是我们java里用的，是底层的机制) ， 写的时候会lock, 写完unlock ，**同时加锁后会将工作内存中的这个变量的值清空，使用的话需要重新load或者assign**。

> 此处没有大面积加锁，只是用了lock ，unlock ，极快的操作，上面的两两都是成对出现的（read-load，use-cpu-assign, store-write, lock-unlock），只有这个use-cpu-assign 会出现cpu处理的时候有间隙，不保证原子性就是这里出现问题了

### 没有原子性

#### volatile变量的复合操作(如i++)不具有原子性

代码验证：

```java
class MyNumber
{
    volatile int number = 0;

    public void addPlusPlus()
    {
        number++;
    }
}

public class VolatileNoAtomicDemo
{
    public static void main(String[] args) throws InterruptedException
    {
        MyNumber myNumber = new MyNumber();

        for (int i = 1; i <=10; i++) {
            new Thread(() -> {
                for (int j = 1; j <= 1000; j++) {
                    myNumber.addPlusPlus();
                }
            },String.valueOf(i)).start();
        }
        
        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println(Thread.currentThread().getName() + "\t" + myNumber.number);
    }
}
 
 

```

![image-20220319210250457](https://image.imxyu.cn/file/image-20220319210250457.png)

按照我们上面读写过程中的图，这三步是不能保证原子性的

![image-20220319210259046](https://image.imxyu.cn/file/image-20220319210259046.png)

> 这个具体后面CAS的时候还会解释，结合那里看更清楚

#### 读取一个普通变量的情况

读取赋值一个**普通变量**（没有标注volatile关键字）的情况：

可以看到线程2会在任意时刻侵入，出现问题

![image-20220319210646686](https://image.imxyu.cn/file/image-20220319210646686.png)



#### 既然修改就可见，为什么不能保证原子性

要use(使用)一个变量的时候必需load(载入），要载入的时候必需从主内存read(读取）这样就解决了读的可见性。 

写操作是把assign和store做了关联(在assign(赋值)后必需store(存储))。store(存储)后write(写入)。  都是两两成对的
也就是做到了给一个变量赋值的时候一串关联指令直接把变量值写到主内存。

就这样通过用的时候直接从主内存取，在赋值到直接写回主内存做到了内存可见性。**注意蓝色框框的间隙。。。。。。**（cpu 处理的这段时间无法保证原子性）



![image-20220319210959846](https://image.imxyu.cn/file/image-20220319210959846.png)



#### 读取一个volatile变量的情况

![image-20220319210821144](https://image.imxyu.cn/file/image-20220319210821144.png)



#### 面试回答

JVM的字节码，i++分成三步，间隙期不同步非原子操作(i++)

![image-20220319211126722](https://image.imxyu.cn/file/image-20220319211126722.png)

### 指令禁重排

![image-20220319211231511](https://image.imxyu.cn/file/image-20220319211231511.png)

![image-20220319211245469](https://image.imxyu.cn/file/image-20220319211245469.png)





#### volatile有关的禁止指令重排的行为

**volatile的底层实现是通过内存屏障，2次复习**

![image-20220319211405452](https://image.imxyu.cn/file/image-20220319211405452.png)

![image-20220319211422101](https://image.imxyu.cn/file/image-20220319211422101.png)



#### code

![image-20220319211444928](https://image.imxyu.cn/file/image-20220319211444928.png)

首先我们分步来看，在flag=true volatile写之前会有一个storestore 的屏障，保证顺序写，如果没有这个保障的话，进行指令重排先flag=true此时其他的flag会立刻更新，此时进行读的话，那么下面read的时候flag=true 对应的i=0

在flag=true 后面会有一个storeload屏障，保证写完了才进行后面的读，如果此时进行指令重排。那么flag还没写，后面读的时候flag=false, i =2。

![image-20220319211551221](https://image.imxyu.cn/file/image-20220319211551221.png)



对于下面这两个，在读之后会有一个loadload 屏障，舍弃本地内存的变量，保证后面读的都要是最新volatile 加载的最新的。

还有一个loadstore, 保证后面写的也是用volatile 读的最新的。（禁止指令重排，否则会出现问题）

![image-20220319212120918](https://image.imxyu.cn/file/image-20220319212120918.png)



## 如何正确使用volatile?

### 1、不能使用复合运算

单一赋值可以，but含复合运算赋值不可以(i++之类)

volatile int a = 10

volatile boolean flag = false



### 2、状态标志，判断业务是否结束

```java
/**
 * @auther zzyy
 * @create 2020-04-14 18:11
 *
 * 使用：作为一个布尔状态标志，用于指示发生了一个重要的一次性事件，例如完成初始化或任务结束
 * 理由：状态标志并不依赖于程序内任何其他状态，且通常只有一种状态转换
 * 例子：判断业务是否结束
 */
public class UseVolatileDemo
{
    private volatile static boolean flag = true;

    public static void main(String[] args)
    {
        new Thread(() -> {
            while(flag) {
                //do something......
            }
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(2L); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            flag = false;
        },"t2").start();
    }
}
 
 
```

### 3、 读+volatile 配合syn写

```java
public class UseVolatileDemo
{
    /**
     * 使用：当读远多于写，结合使用内部锁和 volatile 变量来减少同步的开销
     * 理由：利用volatile保证读取操作的可见性；利用synchronized保证复合操作的原子性
     */
    public class Counter
    {
        private volatile int value;

        public int getValue()
        {
            return value;   //利用volatile保证读取操作的可见性
         }
        public synchronized int increment()
        {
            return value++; //利用synchronized保证复合操作的原子性
         }
    }
}
 
```

可以看到因为上面value 用到了value++ ，volatile 不能保证复合操作的原子性，所以使用复合操作的时候配合synchronized，而读的时候利用volatile，则不用加锁，减少同步的消耗



### 4、DCL双端锁（单例模式问题）

#### volatile

问题：

```java
/**
 * @auther zzyy
 * @create 2020-07-13 15:14
 */
public class SafeDoubleCheckSingleton
{
    private static SafeDoubleCheckSingleton singleton;
    //私有化构造方法
    private SafeDoubleCheckSingleton(){
    }
    //双重锁设计
    public static SafeDoubleCheckSingleton getInstance(){
        //在外面加是为了减少锁的争抢判断，如果为null再进去，抢锁。否则直接返回对象
        if (singleton == null){
            //1.多线程并发创建对象时，会通过加锁保证只有一个线程能创建对象
            synchronized (SafeDoubleCheckSingleton.class){
                //里面这个if 是防止同时进入了外面这层if判断，在锁外面和外层if之间
                if (singleton == null){
                    //隐患：多线程环境下，由于重排序，该对象可能还未完成初始化就被其他线程读取
                    singleton = new SafeDoubleCheckSingleton();
                }
            }
        }
        //2.对象创建完毕，执行getInstance()将不需要获取锁，直接返回创建对象
        return singleton;
    }
}
 
 
```



单线程情况下是没有问题的，有hanppens-before 的保证

![image-20220320094205071](https://image.imxyu.cn/file/image-20220320094205071.png)



而多线程情况下，由于存在指令重排序......

![image-20220320094222408](https://image.imxyu.cn/file/image-20220320094222408.png)



现在因为2和3没有关联依赖性，所以可能会发生指令重排，而重排后对象还没初始化，3直接指向了这个内存地址。

这就会导致下面使用这个instance的时候是个null， 然后初始化对象在这个分配的空间后。又能正确得到这个对象

> 中间会有那么一刻得到的是instance引用为null



![image-20220320094712194](https://image.imxyu.cn/file/image-20220320094712194.png)

使用volatile 关键字解决：

![image-20220320095055715](https://image.imxyu.cn/file/image-20220320095055715.png)



以上是周志明老师在jvm  第三版中提出的解决方案，那么还有其他解决吗？

#### 静态内部类

使用静态内部类的方式，因为静态变量只有一个。所以只会有一个对象的创建

```java
//现在比较好的做法就是采用静态内部内的方式实现
 
public class SingletonDemo
{
    private SingletonDemo() { }

    private static class SingletonDemoHandler
    {
        private static SingletonDemo instance = new SingletonDemo();
    }

    public static SingletonDemo getInstance()
    {
        return SingletonDemoHandler.instance;
    }
}
 
```

## 最后的总结：

打开脑图看

![image-20220320100809142](https://image.imxyu.cn/file/image-20220320100809142.png)



