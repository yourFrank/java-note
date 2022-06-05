# ThreadLocal简介



学完了你应该能轻松回答下面的问题

![image-20220320191259761](https://image.imxyu.cn/file/image-20220320191259761.png)



## ThreadLocal是什么？

![image-20220320191331142](https://image.imxyu.cn/file/image-20220320191331142.png)



## ThreadLocal能干嘛？

![image-20220320191351561](https://image.imxyu.cn/file/image-20220320191351561.png)



实现每一个线程都有自己专属的本地变量副本（自己用自己的变量，不麻烦别人，不和其他人共享，人人有份，人各一份）

![image-20220320191504935](https://image.imxyu.cn/file/image-20220320191504935.png)



> 相当于从锅里拿到自己的碗里，不需要再从锅里拿了，以后就操作自己的碗就行了



![image-20220320191917481](https://image.imxyu.cn/file/image-20220320191917481.png)

## api介绍

这个api中创建ThreadLocal 有两个操作，一个是initialVlue 里面传一个匿名内部类覆盖initValue 方法 , 另一个是静态方法withIntial 传入一个supply 函数式接口 定义初始值

推荐使用静态的withIntial 初始化，代码简洁美观

![image-20220320193938835](https://image.imxyu.cn/file/image-20220320193938835.png)



## hellworld

首先我们来看一个这样一个需求：

三个售票员卖完50张票务，总量完成即可，吃大锅饭，售票员每个月固定月薪

```java


class MovieTicket
{
    int number = 50;

    public synchronized void saleTicket()
    {
        if(number > 0)
        {
            System.out.println(Thread.currentThread().getName()+"\t"+"号售票员卖出第： "+(number--));
        }else{
            System.out.println("--------卖完了");
        }
    }
}

/**
 * @auther zzyy
 * @create 2021-03-23 15:03
 * 三个售票员卖完50张票务，总量完成即可，吃大锅饭，售票员每个月固定月薪
 */
public class ThreadLocalDemo
{
    public static void main(String[] args)
    {
        MovieTicket movieTicket = new MovieTicket();

        for (int i = 1; i <=3; i++) {
            new Thread(() -> {
                for (int j = 0; j <20; j++) {
                    movieTicket.saleTicket();
                    try { TimeUnit.MILLISECONDS.sleep(10); } catch (InterruptedException e) { e.printStackTrace(); }
                }
            },String.valueOf(i)).start();
        }
    }
}
 
 

```

现在我们来看另外一个需求：

卖房子，分提成。 按照每个人卖的房子的多少给提成，不是固定工资了

那就要用到Threadlocal 每个线程独自一份变量，这东西肯定不能共享。 是作为私有的

```java
package com.atguigu.juc.tl;


class MovieTicket
{
    int number = 50;

    public synchronized void saleTicket()
    {
        if(number > 0)
        {
            System.out.println(Thread.currentThread().getName()+"\t"+"号售票员卖出第： "+(number--));
        }else{
            System.out.println("--------卖完了");
        }
    }
}

//卖房子
class House
{
    //推荐使用withInitial的方式创建
    ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);

    public void saleHouse()
    {
        Integer value = threadLocal.get();
        value++;
        threadLocal.set(value);
    }
}

/**
 * @auther zzyy
 * @create 2021-03-23 15:03
 * 1  三个售票员卖完50张票务，总量完成即可，吃大锅饭，售票员每个月固定月薪
 *
 * 2  分灶吃饭，各个销售自己动手，丰衣足食
 */
public class ThreadLocalDemo
{
    public static void main(String[] args)
    {
        /*MovieTicket movieTicket = new MovieTicket();

        for (int i = 1; i <=3; i++) {
            new Thread(() -> {
                for (int j = 0; j <20; j++) {
                    movieTicket.saleTicket();
                    try { TimeUnit.MILLISECONDS.sleep(10); } catch (InterruptedException e) { e.printStackTrace(); }
                }
            },String.valueOf(i)).start();
        }*/

        //===========================================
        House house = new House();

        new Thread(() -> {
            try {
                for (int i = 1; i <=3; i++) {
                    house.saleHouse();
                }
                System.out.println(Thread.currentThread().getName()+"\t"+"---"+house.threadLocal.get());
            }finally {
                house.threadLocal.remove();//如果不清理自定义的 ThreadLocal 变量，可能会影响后续业务逻辑和造成内存泄露等问题
            }
        },"t1").start();

        new Thread(() -> {
            try {
                for (int i = 1; i <=2; i++) {
                    house.saleHouse();
                }
                System.out.println(Thread.currentThread().getName()+"\t"+"---"+house.threadLocal.get());
            }finally {
                house.threadLocal.remove();
            }
        },"t2").start();

        new Thread(() -> {
            try {
                for (int i = 1; i <=5; i++) {
                    house.saleHouse();
                }
                System.out.println(Thread.currentThread().getName()+"\t"+"---"+house.threadLocal.get());
            }finally {
                house.threadLocal.remove();
            }
        },"t3").start();


        System.out.println(Thread.currentThread().getName()+"\t"+"---"+house.threadLocal.get());
    }
}
 
 

```



> 注意上面因为用到了threadLocal .set（） 方法 ，所以最后一定要house.threadLocal.remove(); **在finally 中用完了一定要加这句** ，因为多线程情况下一般会用到线程池，**线程会被复用**，如果不清理自定义的ThreadLocal 变量，可能会影响后续的逻辑造成内存泄漏，OOM 。 所以用完了一定要remove删除



# 从阿里ThreadLocal规范开始（你项目出现过什么问题？）

非线程安全的SimpleDateFormat

![image-20220320201149678](https://image.imxyu.cn/file/image-20220320201149678.png)

## 问题代码

话术： 我之前刚到公司，要求根据前端传入的日历格式进行转换写到数据库（写日志等）中，然后我就使用SimpleDateFormat ，但是每次都要new 很麻烦，我就直接封装成静态的然后给其他人使用，觉得干了个好事， 结果很多人一起用然后出现各种报错，然后我就在网上学习，发现这个东西是非线程安全的，底层有一个Calender引用，多个线程的话会共享这个引用。解决方案: 使用ThreadLocal，然后展开说

![image-20220320203142757](https://image.imxyu.cn/file/image-20220320203142757.png)

```java
public class DateUtils
{
    public static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    /**
     * 模拟并发环境下使用SimpleDateFormat的parse方法将字符串转换成Date对象
     * @param stringDate
     * @return
     * @throws Exception
     */
    public static Date parseDate(String stringDate)throws Exception
    {
        return sdf.parse(stringDate);
    }
    
    public static void main(String[] args) throws Exception
    {
        for (int i = 1; i <=30; i++) {
            new Thread(() -> {
                try {
                    System.out.println(DateUtils.parseDate("2020-11-11 11:11:11"));
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
 

```

报错：

![image-20220320201245422](https://image.imxyu.cn/file/image-20220320201245422.png)



## 源码分析问题

![image-20220320201530442](https://image.imxyu.cn/file/image-20220320201530442.png)

![image-20220320201608461](https://image.imxyu.cn/file/image-20220320201608461.png)

![image-20220320201612824](https://image.imxyu.cn/file/image-20220320201612824.png)

## 解决方案

1、将SimpleDateFormat定义成局部变量。（不好，这样的话线程多了会造成jvm垃圾回收的负担new了很多对象，cpu打满）

缺点：每调用一次方法就会创建一个SimpleDateFormat对象，方法结束又要作为垃圾回收。 

```java
/**
 * @auther zzyy
 * @create 2020-07-17 16:42
 */
public class DateUtils
{
    public static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    /**
     * 模拟并发环境下使用SimpleDateFormat的parse方法将字符串转换成Date对象
     * @param stringDate
     * @return
     * @throws Exception
     */
    public static Date parseDate(String stringDate)throws Exception
    {
        return sdf.parse(stringDate);
    }

    public static void main(String[] args) throws Exception
    {
        for (int i = 1; i <=30; i++) {
            new Thread(() -> {
                try {
                    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                    System.out.println(sdf.parse("2020-11-11 11:11:11"));
                    sdf = null;
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```



2、使用ThreadLocal，也叫做线程本地变量或者线程本地存储 。每个线程用自己的一份

```java
package com.atguigu.itdachang;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @auther zzyy
 * @create 2020-07-17 16:42
 */
public class DateUtils
{
    private static final ThreadLocal<SimpleDateFormat>  sdf_threadLocal =
            ThreadLocal.withInitial(()-> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

    /**
     * ThreadLocal可以确保每个线程都可以得到各自单独的一个SimpleDateFormat的对象，那么自然也就不存在竞争问题了。
     * @param stringDate
     * @return
     * @throws Exception
     */
    public static Date parseDateTL(String stringDate)throws Exception
    {
        return sdf_threadLocal.get().parse(stringDate);
    }

    public static void main(String[] args) throws Exception
    {
        for (int i = 1; i <=30; i++) {
            new Thread(() -> {
                try {
                    System.out.println(DateUtils.parseDateTL("2020-11-11 11:11:11"));
                } catch (Exception e) {
                    e.printStackTrace();
                } finally{
                    DateUtils.sdf_threadLocal.remove();  //一定不要忘了这句
                }
            },String.valueOf(i)).start();
        }
    }
}
 

```

> 最后要remove 掉这个threadlocal，记得！



3、直接在方法加锁Synchcronized 或者使用第3方时间库



## DateUtils 工具类的正确编写（遵循阿里手册）

```java
/**
 * @auther zzyy
 * @create 2020-05-03 10:14
 */
public class DateUtils
{
    /*
    1   SimpleDateFormat如果多线程共用是线程不安全的类（以前错误的写法）
    public static final SimpleDateFormat SIMPLE_DATE_FORMAT = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static String format(Date date)
    {
        return SIMPLE_DATE_FORMAT.format(date);
    }

    public static Date parse(String datetime) throws ParseException
    {
        return SIMPLE_DATE_FORMAT.parse(datetime);
    }*/

    //2   ThreadLocal可以确保每个线程都可以得到各自单独的一个SimpleDateFormat的对象，那么自然也就不存在竞争问题了。
    public static final ThreadLocal<SimpleDateFormat> SIMPLE_DATE_FORMAT_THREAD_LOCAL = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

    public static String format(Date date)
    {
        return SIMPLE_DATE_FORMAT_THREAD_LOCAL.get().format(date);
    }

    public static Date parse(String datetime) throws ParseException
    {
        return SIMPLE_DATE_FORMAT_THREAD_LOCAL.get().parse(datetime);
    }


    //3 DateTimeFormatter 代替 SimpleDateFormat （最好是这种）
    /*public static final DateTimeFormatter DATE_TIME_FORMAT = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    public static String format(LocalDateTime localDateTime)
    {
        return DATE_TIME_FORMAT.format(localDateTime);
    }

    public static LocalDateTime parse(String dateString)
    {

        return LocalDateTime.parse(dateString,DATE_TIME_FORMAT);
    }*/
}
 
```

# ThreadLocal源码分析

源码解读

调用threadlocal.get（）方法会调用getMap , 如果map 为空的话会创建一个map，否则map 中获取这个值（k,v键值对的形式， key 就是当前的线程）

![image-20220320210907226](https://image.imxyu.cn/file/image-20220320210907226.png)



![image-20220320210855969](https://image.imxyu.cn/file/image-20220320210855969.png)

如果map 为null创建初始化map的代码。 

![image-20220320211011239](https://image.imxyu.cn/file/image-20220320211011239.png)

![image-20220320211143650](https://image.imxyu.cn/file/image-20220320211143650.png)



set（）方法：

也是一样的，得到map， 然后设置值 ，key为当前的ThreadLocal，值为要设置的值

map 不存在的话就创建

![image-20220320211839597](https://image.imxyu.cn/file/image-20220320211839597.png)

## Thread，ThreadLocal，ThreadLocalMap 关系（面试题）

![image-20220320211214209](https://image.imxyu.cn/file/image-20220320211214209.png)

![image-20220320211221249](https://image.imxyu.cn/file/image-20220320211221249.png)

![image-20220320211229061](https://image.imxyu.cn/file/image-20220320211229061.png)





![image-20220320211410990](https://image.imxyu.cn/file/image-20220320211410990.png)



>  可以近似的理解为：Thread 相当于一个成年人，要去ThreadLocal 这个派出所，申请ThreadLocalMap这样一个身份证， 身份证的卡号是entry

## 小总结

![image-20220320211302362](https://image.imxyu.cn/file/image-20220320211302362.png)

也就是说一个Thread对应一个ThreadLocal，而ThreadLocal中有一个ThreadLocalMap , 这个Map中根据KEY：ThreadLocal， value：值， 因此每个Thread可以找到当前Threadlocal中保存的值（一个ThreadLocal只能保存一个值）



# ThreadLocal内存泄露问题（之后的看脑图）