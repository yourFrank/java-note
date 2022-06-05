

# 多线程复习

## (面试题)为什么多线程极其重要？

**1、硬件方面：摩尔定律失效**

摩尔定律：
它是由英特尔创始人之一Gordon Moore(戈登·摩尔)提出来的。其内容为：当价格不变时，集成电路上可容纳的元器件的数目约每隔18-24个月便会增加一倍，性能也将提升一倍。换言之，每一美元所能买到的电脑性能，将每隔18-24个月翻一倍以上。这一定律揭示了信息技术进步的速度。

可是从2003年开始CPU主频已经不再翻倍（板子已经插满了，此时只能增加核数），而是采用**多核**而不是更快的主频。

摩尔定律失效。

在**主频不再提高**且**核数在不断增加**的情况下，要想让程序更快就要用到**并行**或**并发编程**。



**2、软件方面**

需要高并发系统，异步+回调等生产需求

充分利用多核数，从硬件到软件的打通



## 从start一个线程说起

### Java中的实现

线程.start(),	调用的就是start0() , 方法

1、private native void start0();   底层是C++写的

查看openJdk 源代码http://openjdk.java.net/  （外网慢，建议下载源码到本地看）

### C++中的解读

1、openjdk8\jdk\src\share\native\java\lang

![image-20220316132638188](https://image.imxyu.cn/file/image-20220316132638188.png)

2、openjdk8\hotspot\src\share\vm\prims

![image-20220316132658047](https://image.imxyu.cn/file/image-20220316132658047.png)

![image-20220316132706707](https://image.imxyu.cn/file/image-20220316132706707.png)

3、openjdk8\hotspot\src\share\vm\runtime

![image-20220316132715974](https://image.imxyu.cn/file/image-20220316132715974.png)

> JNI ： java native interface



## java 多线程相关概念



### 进程

是程序的⼀次执⾏，是系统进⾏资源分配和调度的独⽴单位，每⼀个进程都有它⾃⼰的内存空间和系统资源

### 线程

在同⼀个进程内⼜可以执⾏多个任务，⽽这每⼀个任务我们就可以看做是⼀个线程

⼀个进程会有1个或多个线程的



### 面试题：何为线程和进程？



> 任意时刻一个单核cpu只能运行一个进程，其他进程处于非运行状态（而我们以前在单核cpu的时候能同时运行多个任务的原因是因为时间片的切换速度很快，实际上还是一个时间只执行了一个进程），进程申请的内存空间其中所有的线程都是共享这些内存的。	----> 1个cpu===运行1个进程====运行多个线程
>
> 在使用某些共享内存时，1次只能一个线程其他线程要等待，解决这个问题就是加锁（互斥锁mutex）----> 防止线程冲突
>
> 而对于有些内存，一次可以给固定数目的内存使用，使用的就是信号量（Semaphore），相当于门口的N把钥匙， 当N=1的时候效果和互斥锁mutex 效果是一样的是可以替代mutex的，但是mutex简单效率高，如果保证资源独占的情况下还是使用Mutex    -----> 允许线程之间共享资源



总结：以操作系统的设计：

1）以多进程的形式，允许多个任务同时执行

2）以多线程的形式，允许单个任务（进程）分成不同的部分（线程）执行

3）**提供协调机制，一方面防止进程之间和线程之间冲突，另一方面允许进程之间和线程之间共享资源**



### 管程

#### Monitor(监视器)，也就是我们平时所说的锁

Monitor其实是一种同步机制，他的义务是保证（同一时间）只有一个线程可以访问被保护的数据和代码。

JVM中同步是基于进入和退出监视器对象(Monitor,管程对象)来实现的，**每个对象实例都会有一个Monitor对象**，

```java
 Object o = new Object();

new Thread(() -> {
    synchronized (o)
    {

	}

},"t1").start();
```



Monitor对象会和Java对象一同创建并销毁，它底层是由C++语言来实现的。



#### 在jvm 虚拟机中：

执行线程就要求先成功持有管程（得到锁），然后才能执行方法，最后当方法执行完成（无论是正常完成还是非正常完成）时都释放管程。在方法执行期间，执行线程持有了管程，其他任何线程都无法再获取到同一个管程



### 用户线程和守护线程

Java线程分为用户线程和守护线程，
线程的daemon属性为true表示是守护线程，false表示是用户线程

守护线程是一种特殊的线程，在后台默默地完成一些系统性的服务，比如垃圾回收线程

用户线程是系统的工作线程，它会完成这个程序需要完成的业务操作

```java
/**
 * @auther zzyy
 * @create 2020-07-07 15:39
 */
public class DaemonDemo
{
public static void main(String[] args)
{
    Thread t1 = new Thread(() -> {
        System.out.println(Thread.currentThread().getName()+"\t 开始运行，"+(Thread.currentThread().isDaemon() ? "守护线程":"用户线程"));
        while (true) {

        }
    }, "t1");
    //线程的daemon属性为true表示是守护线程，false表示是用户线程
    t1.setDaemon(true);
    t1.start();
    //3秒钟后主线程再运行
    try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }

    System.out.println("----------main线程运行完毕");
}

}

```

> 可以通过daemon 设置为用户线程还是守护线程

可以看到当主线程运行结束之后，还在运行的守护线程也会自动结束

![image-20220316150434781](https://image.imxyu.cn/file/image-20220316150434781.png)

1、**当程序中所有用户线程执行完毕之后，不管守护线程是否结束，系统都会自动退出**

**如果用户线程全部结束了，意味着程序需要完成的业务操作已经结束了，系统可以退出了。所以当系统只剩下守护进程的时候，java虚拟机会自动退出**



2、**设置守护线程，需要在start()方法之前进行**

```java
t1.start(); 
t1.setDaemon(true);
//这样的顺序是错误的
```


# 引入CompletableFuture

## 实现线程的几种方式

1）继承Thread类创建线程

2）实现Runnable接口创建线程

3）通过Callable和FutureTask创建线程

4）使用线程池例如用Executor框架

实现Runnable和实现Callable接口的方式基本相同，不过是后者执行call()方法有返回值。



## 之前的Future接口

**Future**接口定义了操作异步任务执行一些方法，如获取异步任务的执行结果、取消任务的执行、判断任务是否被取消、判断任务执行是否完毕等。

比如主线程让一个子线程去执行任务，子线程可能比较耗时，启动子线程开始执行任务后，主线程就去做其他事情了，过了一会才去获取子任务的执行结果。



**Callable**接口中定义了需要有返回的任务需要实现的方法, 并且需要一个返回值（作为方法的返回值）

![image-20220316190959817](https://image.imxyu.cn/file/image-20220316190959817.png)



### FutreTask

首先我们来看一下Future接口的实现类FutureTask：



![image-20220316152034121](https://image.imxyu.cn/file/image-20220316152034121.png)

FutureTask需要传入一个Callable 接口。在Callable 接口中可以定义我们任务执行完成后的返回值，后面可以得到这个返回值

![image-20220316191130514](https://image.imxyu.cn/file/image-20220316191130514.png)



现在我们要实现一个需求：

主线程向下执行(老师继续上课)，同时会起一个分线程去执行其他任务（找班长去买水），分线程执行完了之后会将相应的结果交给主线程，同时在分线程运行的过程中不会影响主线程的运行，两者是异步同时向下执行

下面我们使用FutureTask来实现这个需求：

```java
public class FutureDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //FutureTask 方法中要传入一个Callable 接口，通过lambda
        FutureTask<Integer> futureTask = new FutureTask<>(() -> {
            System.out.println("futuretask come in ----");
            TimeUnit.SECONDS.sleep(3);
            return 1024;
        });

        Thread t1=new Thread(futureTask,"t1");
        t1.start();
        System.out.println("我继续上课");
        System.out.println(futureTask.get());
    }
}
```

执行结果：

![image-20220316154400121](https://image.imxyu.cn/file/image-20220316154400121.png)





### FutrueTask的缺点：

**get()阻塞，一旦调用get()方法，不管是否计算完成都会导致阻塞**

因此这个get()方法如果放到上面，则下面的都要等get() 得到结果之后才会执行

![image-20220316154526964](https://image.imxyu.cn/file/image-20220316154526964.png)

> 千万不能在程序中直接使用get()方法，否则会因为你的代码拖慢程序

### 如何避免get（）阻塞？

0、最简单的就是把get （）放到代码最后去获取结果

但是如果我们非要把get（）放到前面呢？ 可以有下面的两种优化的方式：

1、首先通过带时间参数的get方法替代

![image-20220316154935862](https://image.imxyu.cn/file/image-20220316154935862.png)



2、**使用轮询**

轮询就是在执行的过程中不停的去询问： 算完没，算完没？

```java
public class FutureDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        FutureTask<Integer> futureTask = new FutureTask<>(() -> {
            System.out.println("futuretask come in ----");
            TimeUnit.SECONDS.sleep(3);
            return 1024;
        });

        Thread t1=new Thread(futureTask,"t1");
        t1.start();

        //使用轮询的方式
        while(true)
        {
            if (futureTask.isDone())
            {
                System.out.println(futureTask.get());
                break;
            }else{
                //可以在这里添加超时退出的逻辑
                
                System.out.println("正在执行，别催");
            }
        }
//        System.out.println(futureTask.get(2l, TimeUnit.SECONDS));
        System.out.println("我继续上课");

    }
}

```

执行结果：

![image-20220316155253640](https://image.imxyu.cn/file/image-20220316155253640.png)

> 但是使用轮询还是会一直阻塞着，即使我们设置了一个小一点的时间让它超时退出，但还是很难受。如果get( )放到前面，我们的主线程还是要等get()阻塞完得到结果之后才能执行



因此现在我们不想使用阻塞，也不想让它轮询，下面我们看看有什么可以替代的？让他俩能同时异步执行

## 功能强大的CompletableFuture

如果我们想完成一些复杂的任务：

1、应对Future的完成时间，完成了可以告诉我，也就是我们的回调通知

2、将两个异步计算合成一个异步计算，这两个异步计算互相独立，同时第二个又依赖第一个的结果。

3、当Future集合中某个任务最快结束时，返回结果。

4、等待Future结合中的所有任务都完成。



那么我就要使用CompletableFuture这个类了，我们先来看一下这个类的结构图



使用接口类**CompletableFuture**，这个接口实现了**Future**，还实现了**CompletionStage**

![image-20220316155735113](https://image.imxyu.cn/file/image-20220316155735113.png)

> 这个类是从java8中引入的
>
> 也就是说这个接口在原Future 接口上还拥有了**CompletionStage**的功能，对Future接口做了增强
>
> 那么我们来看看什么是CompletionStage？

### 接口CompletionStage

介绍：

![image-20220316155913070](https://image.imxyu.cn/file/image-20220316155913070.png)

类似 linux 中的 ps -ef| grep java | wc -l

> 可以是前一个命令的结果传给下一个这样依次执行下去，也可以是这三个命令一起执行最后汇总成一个结果返回

### CompletableFuture的四个静态方法

现在的**CompletableFuture**作为CompletionStage实现类，会拥有**CompletionStage**的功能还有拥有**future**的功能 ，我们来看一下介绍：

![image-20220316160304802](https://image.imxyu.cn/file/image-20220316160304802.png)

下面我们来看一下CompletableFuture的四个核心静态方法

1、**runAsync 无 返回值**

public static CompletableFuture<Void> runAsync(Runnable runnable)  //使用默认线程池

public static CompletableFuture<Void> runAsync(Runnable runnable,Executor executor)  //指定线程池

2、**supplyAsync 有 返回值**

public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)

public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,Executor executor)



上述Executor executor参数说明

没有指定Executor的方法，直接使用默认的ForkJoinPool.commonPool() 作为它的线程池执行异步代码。

如果指定线程池，则使用我们自定义的或者特别指定的线程池执行异步代码



下面用代码来看一下这四个方法

```java
  public static void main(String[] args) throws ExecutionException, InterruptedException {

        //创建一个线程池
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(1, 20, 1L, TimeUnit.SECONDS, new LinkedBlockingDeque<>(50), Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());
        CompletableFuture<Void> c1 = CompletableFuture.runAsync(() -> {
            System.out.println(Thread.currentThread().getName()+"无返回值，不指定线程池");
        });
        //我们说过会CompletableFuture会有future 的功能， 我们直接调用get ()方法获取返回值
        System.out.println(c1.get());
        CompletableFuture<Void> c2 = CompletableFuture.runAsync(() -> {
            System.out.println(Thread.currentThread().getName()+"无返回值，不指定线程池");
        },threadPoolExecutor);

        System.out.println(c2.get());
        threadPoolExecutor.shutdown();
    }
```

可以看到，调用runAsync 是没有返回值的。Void 类型表示没有返回值

并且使用默认线程池的是的名字是ForkJoinPool

![image-20220316182141920](https://image.imxyu.cn/file/image-20220316182141920.png)

接下来我们看一下那两个有返回值的，参数要求我们提供一个supplier接口

调用supplyAsync需要我们指定返回值的

```java
   CompletableFuture<Integer> c3 = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "有返回值，不指定线程池");
            return 1;
        });

        System.out.println(c3.get());
        CompletableFuture<Integer> c4 = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "有返回值，指定线程池");
            return 2;
        },threadPoolExecutor);
        System.out.println(c4.get());
```

可以看到这四个方法所对应的输出如下：

![image-20220316182525297](https://image.imxyu.cn/file/image-20220316182525297.png)



## 使用CompletableFuture完成异步调用（解决futureTask阻塞）



我们说过CompletableFuture实现了future接口，因此还是可以使用future 中的功能，

因此我们还是可以使用future 的get() 方法，可以看到下面代码，那就和之前的futureTask 没区别了，还是会get（）阻塞。

下面我们主要来看一下CompletableFuture的强大功能。

![image-20220316190500011](https://image.imxyu.cn/file/image-20220316190500011.png)



上面我们说了使用CompletableFuture可以完成多个异步操作，并且不同的异步操作之间可以依赖结果，然后最终返回一个结果回去

下面我们就来体验一下

以下都是通过链式调用+lambda 函数式接口的方式

```java
  public static void main(String[] args) throws ExecutionException, InterruptedException {

        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(1, 20, 1L, TimeUnit.SECONDS, new LinkedBlockingDeque<>(50), Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());
          CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "有返回值的，不指定线程池");
            return 1;
        }).thenApply((r)->{
            return r+3;   //把上一次的结果拿过来，进行处理
        }).whenComplete((res,ex)->{  //完成了以后主动来告诉主线程，这个whenComplete参数是BiConsumer的接口，传两个值（结果和异常）
            if(ex==null){ //如果没有异常
                System.out.println(res);   //输出结果
            }
        }).exceptionally(e->{	
            e.printStackTrace();  //有异常打印异常
            return null;
        });

        System.out.println("main ---- over");

        TimeUnit.SECONDS.sleep(5);  //主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:暂停5秒钟线程
      	//或者我们自己带一个线程池，使用带线程池参数的那个supplyAsync方法

    }
```

运行结果

![image-20220316185315352](https://image.imxyu.cn/file/image-20220316185315352.png)

> 上面使用的thenApply 是等待上一个结果执行完，拿到结果后才执行的

## CompletableFuture的优点

CompletableFuture相当于Future的增强版

通过上面的案例我们可以看到CompletableFuture的强大之处：

1、异步任务结束时，会自动回调某个对象的方法；

2、异步任务出错时，会自动回调某个对象的方法；

3、主线程设置好回调后，不再关心异步任务的执行，异步任务之间可以顺序执行

由之前的 1 =》  2 =》 3 =》4		step by step 变成，四步可以同时执行



## 函数式编程回顾

![image-20220316192909060](https://image.imxyu.cn/file/image-20220316192909060.png) 

## join()和get() 方法的区别

join 和get 方法一样都会阻塞，唯一不同的就是join 方法不会抛出异常

以后推荐使用join（）方法，少抛点异常系统健壮一点点

get（）得到结果:

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {

        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(1, 20, 1L, TimeUnit.SECONDS, new LinkedBlockingDeque<>(50), Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {

            return 1;
        }).whenComplete((res, ex) -> {
            if (ex == null) {
                System.out.println("result:" + res);
            }
        }).exceptionally(e -> {
            e.printStackTrace();  //有异常打印异常
            return null;
        });

        System.out.println(completableFuture.get());
    }

```

可以直接函数式调用，一波带走

![image-20220316194124380](https://image.imxyu.cn/file/image-20220316194124380.png)



join() ：

使用join 方法，可以看到。不需要抛出异常了

![image-20220316194156097](https://image.imxyu.cn/file/image-20220316194156097.png)



## 案例精讲：写到简历上的比价需求



案例说明：电商比价需求

1、同一款产品，同时搜索出本款产品在各大电商的售价；

2、同一款产品，同时搜索出本产品在某一个电商平台下， 各个入驻门店的售价是多少

出来的结果是在同款产品在不同地方的价格清单列表，返回一个List<String>

《mysql》 in jd  prcice is 88.05

《mysql》 in pdd prcice is 88.05

《mysql》 in taobao prcice is 88.05

```java
public class CompletableFutureNetMallDemo {

    static List<NetMall> list= Arrays.asList(
        new NetMall("jd"),
        new NetMall("taobao"),
        new NetMall("pdd"),
         new NetMall("dangdangwang") ,
         new NetMall("tmall")
    );


    //同步获取价格的方法
    public static List<String> findPriceSync(List<NetMall> list,String productName){
        return list.stream().map(netMall -> String.format(productName + " %s price is %.2f", netMall.getMallName(), netMall.getPrice(productName))).collect(Collectors.toList());

    }
    

    /**
     *异步，多箭齐发
     * List<NetMall> ----> List<CompletableFuture<String>>------> List<String>
     * @param list
     * @param productName
     * @return
     */
    public static List<String> findPriceASync(List<NetMall> list,String productName){

        return list.stream().map(netMall -> CompletableFuture.supplyAsync(() -> String.format(productName + " %s price is %.2f", netMall.getMallName(), netMall.getPrice(productName)))).collect(Collectors.toList()).stream().map(CompletableFuture::join).collect(Collectors.toList());

    }

    public static void main(String[] args) {
        long startTime = System.currentTimeMillis();
        List<String> list1 = findPriceSync(list, "thinking in java");
        for (String element : list1) {
            System.out.println(element);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("----costTime: "+(endTime - startTime) +" 毫秒");

        long startTime2 = System.currentTimeMillis();
        List<String> list2 = findPriceASync(list, "thinking in java");
        for (String element : list2) {
            System.out.println(element);
        }
        long endTime2 = System.currentTimeMillis();
        System.out.println("----costTime: "+(endTime2 - startTime2) +" 毫秒");

    }
}

/**
 * 资源类，电商
 */
class  NetMall{

    @Getter
    private String mallName;

    public NetMall(String mallName) {
        this.mallName = mallName;
    }

    //模拟计算价格（可以从redis中取）
    public double getPrice(String bookName){
        //检索需要1秒钟
        try {   TimeUnit.SECONDS.sleep(1);   } catch (InterruptedException e) { e.printStackTrace(); }
        return ThreadLocalRandom.current().nextDouble()+ bookName.charAt(0);
    }

}

```

运行结果：

![image-20220318151339490](https://image.imxyu.cn/file/image-20220318151339490.png)



### 话术

我做过一个比价需求，从其他网站上得到一件物品的在不同厂商卖的价格（比如说京东对mysql这本书有不同厂家在卖，价格不同），我上一家是爬虫兄弟它负责爬取数据，给了我一堆json数据，我把数据去重之后放到redis的zset 里面 中保证数据干净没有重复，大概有一万条数据

一开始想的方案是一个一个去获得相应的数据可以做但是性能不高，后来我在Juc 中发现有一个叫CompletableFuture 是做多线程并发的，不阻塞，成功的将数据从7s 变成了 1.5s ,   CompletableFuture 默认是用ForkJoinPool 线程池，而我自己实现了线程池 速度可以自由伸缩，

把网站从同步变成了异步编排，性能上有极具提升

![image-20220318145825077](https://image.imxyu.cn/file/image-20220318145825077.png)



### 链式调用

一套带走

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true)
public class Book {
    private Integer id;
    private String name;
    private double price;
    private String author;

    public static void main(String[] args) {
        Book book=new Book();
        //以前的写法
//        book.setAuthor("张三");
//        book.setId(1);
//        book.setPrice(50l);
        
        
        //加了@Accessors(chain = true) 后可以使用链式调用，这个默认为false
        book.setId(1).setName("张三").setPrice(50l);
                
    }
}


```



## CompletableFuture 常用方法 

### 1、获取结果和触发方法

#### 获取结果

![image-20220318152701122](https://image.imxyu.cn/file/image-20220318152701122.png)



get()方法会一直阻塞

public T    get(long timeout, TimeUnit unit)   会根据给的时间，如果给定时间内没有算完，直接抛出异常

public T    getNow(T valueIfAbsent)  上面两个的折中方法，如果不想阻塞，还不想抛出异常。 getNow (T) 如果计算完了可以直接得到值，否则使用我的替代值

join() 和get（） 上面说过了，join 不会抛出异常

#### 主动触发计算

是否打断get方法立即返回括号值 ，这个和上面的getNow 方法很像，这个返回的是boolean类型的。需要再调取get获取一下值

public boolean complete(T value) 



```java
public class CompletableFutureDemo2
{
    public static void main(String[] args) throws ExecutionException, InterruptedException
    {
        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(() -> {
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            return 533;
        });

        //注释掉暂停线程，get还没有算完只能返回complete方法设置的444；暂停2秒钟线程，异步线程能够计算完成返回get
        try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }

        //当调用CompletableFuture.get()被阻塞的时候,complete方法就是结束阻塞并get()获取设置的complete里面的值.
        System.out.println(completableFuture.complete(444)+"\t"+completableFuture.get());


    }
}
 
```

结果：

![image-20220318152947488](https://image.imxyu.cn/file/image-20220318152947488.png)



如果已经计算完了，就用你的值

![image-20220318153012258](https://image.imxyu.cn/file/image-20220318153012258.png)

结果

![image-20220318153034478](https://image.imxyu.cn/file/image-20220318153034478.png)

### 2、对计算结果进行处理

#### thenApply

计算结果存在依赖关系，由于存在依赖关系(当前步错，不走下一步)，当前步骤有异常的话就叫停。

```java
public static void main(String[] args) throws ExecutionException, InterruptedException
{
    //当一个线程依赖另一个线程时用 thenApply 方法来把这两个线程串行化,
    CompletableFuture.supplyAsync(() -> {
        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("111");
        return 1024;
    }).thenApply(f -> {
         //int age = 10/0; // 异常情况：那步出错就停在那步。
        System.out.println("222");
        return f + 1;
    }).thenApply(f -> {
        System.out.println("333");
        return f + 1;
    }).whenCompleteAsync((v,e) -> {
        System.out.println("*****v: "+v);
    }).exceptionally(e -> {
        e.printStackTrace();
        return null;
    });

    System.out.println("-----主线程结束，END");

    // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
    try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
}

```

下面的一步可以得到上一步的结果



如果我们故意制造一个异常的话：

可以看到只会执行111， 后面的222和333都不会执行了

![image-20220318155036743](https://image.imxyu.cn/file/image-20220318155036743.png)



#### handle

和thenApply 的区别是：有异常也可以往下一步走，根据带的异常参数可以进一步处理

```java
public static void main(String[] args) throws ExecutionException, InterruptedException
    {
        //当一个线程依赖另一个线程时用 handle 方法来把这两个线程串行化,
        // 异常情况：有异常也可以往下一步走，根据带的异常参数可以进一步处理
        CompletableFuture.supplyAsync(() -> {
            //暂停几秒钟线程
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println("111");
            return 1024;
        }).handle((f,e) -> {
            int age = 10/0;
            System.out.println("222");
            return f + 1;
        }).handle((f,e) -> {
            System.out.println("333");
            return f + 1;
        }).whenCompleteAsync((v,e) -> {
            System.out.println("*****v: "+v);
        }).exceptionally(e -> {
            e.printStackTrace();
            return null;
        });

        System.out.println("-----主线程结束，END");

        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
        try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
    }
 
```

执行结果：

![image-20220318154746179](https://image.imxyu.cn/file/image-20220318154746179.png)



#### 总结

![image-20220318154402117](https://image.imxyu.cn/file/image-20220318154402117.png)



### 带有Async的方法和普通的区别

所有后面带Async的方法，意思都是重新交给线程池分配线程，并不是说又起了另一个线程

比如说当前线程执行了70%到这， 遇到Async的方法会让线程池重新分配一个线程执行剩下的百分之30

![image-20220318154411224](https://image.imxyu.cn/file/image-20220318154411224.png)

> 直接使用普通的就行，使用Async 还要等待线程池重新分配有时间片的消耗



### 3、对计算结果进行消费

这几个方法都是对计算结果进行消费的

通过他们之间需要传入的函数式接口也可以看出来，能否有参数，能否有返回

![image-20220318155621033](https://image.imxyu.cn/file/image-20220318155621033.png)



代码：

![image-20220318155749230](https://image.imxyu.cn/file/image-20220318155749230.png)



### 4、计算速度优选applyToEither

谁快选谁，applyToEither

applyToEither 中的参数一个为比较的线程，另一个函数式接口为得到的结果

![image-20220318160947232](https://image.imxyu.cn/file/image-20220318160947232.png)

类似于游戏中的对战，两个人比拼俄罗斯方块，谁先结束（谁快）谁赢（选谁）。

![image-20220318160359254](https://image.imxyu.cn/file/image-20220318160359254.png)

completableFuture1.applyToEither(completableFuture2, 结果)



![image-20220318161259629](https://image.imxyu.cn/file/image-20220318161259629.png)



### 5、对计算结果进行合并thenCombine

两个CompletionStage任务都完成后，最终能把两个任务的结果一起交给thenCombine 来处理

先完成的先等着，等待其它分支任务，所有都执行完了一起合并

![image-20220318163213598](https://image.imxyu.cn/file/image-20220318163213598.png)

参数1：第二个CompletionStage， 参数2： CompletionStage1和CompletionStage2的结果进行的操作（BiFunction）



#### 拆分版，比较好理解：

![image-20220318163027126](https://image.imxyu.cn/file/image-20220318163027126.png)



#### 一波流

```java
 public static void main(String[] args) throws ExecutionException, InterruptedException
    {
        CompletableFuture<Integer> thenCombineResult = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "\t" + "---come in 1");
            return 10;
        }).thenCombine(CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "\t" + "---come in 2");
            return 20;
        }), (x,y) -> {
            return x + y;
        }).thenCombine(CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "\t" + "---come in 3");
            return 30;
        }),(a,b) -> {
            return a + b;
        });
        System.out.println("-----主线程结束，END");
        System.out.println(thenCombineResult.get());


        // 主线程不要立刻结束，否则CompletableFuture默认使用的线程池会立刻关闭:
        try { TimeUnit.SECONDS.sleep(10); } catch (InterruptedException e) { e.printStackTrace(); }
    }
```

结果：

![image-20220318163800154](https://image.imxyu.cn/file/image-20220318163800154.png)

> 可以看到我们可以进行多次组合，和多个CompletionStage进行组合，最后一起合并。非常nb



应用场景：微服务的两个模块（多个模块）的聚合，两个模块分别执行，然后将得到的结果进行聚合

（数据中台，将多个数据加工之后封装成一个公共的数据产品或服务）

### 6、流水线thenCompose

和thenApply的功能上类似的，都是将上一个结果传给下一个。不同的是thenCompose 中可以继续使用异步CompletableFuture操作作为返回值，而thenapply则是直接会返回一个对象

```java
<U> CompletionStage<U> thenApply(Function<? super T,? extends U> fn)

<U> CompletionStage<U> thenCompose(Function<? super T,? extends CompletionStage<U>> fn)
```

```java
        CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> 100).thenApply(num -> num + " to String");
        CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> 100).thenCompose(num -> CompletableFuture.supplyAsync(() -> num + " to String"));

        System.out.println(f1.join()); // 100 to String
        System.out.println(f2.join()); // 100 to String
```

例子中，`thApply`和`thenCompose`都是将一个`CompletableFuture<Integer>`转换为`CompletableFuture<String>`。不同的是，`thenApply`中的传入函数的返回值是`String`，而`thenCompose`的传入函数的返回值是`CompletionStage<String>`。就好像stream中学到的`map`和`flatMap`。回想我们做过的二维数组转一维数组，使用`stream().flatMap`映射时，我们是把流中的每个数据（数组）又展开为了流。



对于这种要求得到前面的结果进行后面的处理的，不会进行异步操作，会等到第一个执行完了拿到值之后才继续进行compose后面的stage中的操作

可以看到第一个需要1秒，第二个需要3秒。 最后的结果是4秒，并没有加快速度，只是为了得到结果进行合并而已

![image-20220319092031055](https://image.imxyu.cn/file/image-20220319092031055.png)