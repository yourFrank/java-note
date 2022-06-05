**JDK体系结构**

------

![image-20220322203050387](https://image.imxyu.cn/file/image-20220322203050387.png)

**Java语言的跨平台特性**

在不同的操作系统上通过下载不同的jdk, jdk 中jre里的jvm的不同，帮我们解析的class文件出来的指令就不同（比如win下0101, linux 下变成1010等），这样就保证了跨平台的特性，底层是我们的jvm做的

------

![image-20220322203059964](https://image.imxyu.cn/file/image-20220322203059964.png)

# **JVM整体结构及内存模型**

------

​    ![image-20220322203225732](https://image.imxyu.cn/file/image-20220322203225732.png)

## 栈

每有一个线程就会在栈上分配一块内存空间，比如有main 线程，就会在栈上为main线程划分出一块内存空间，用来装每个线程的变量、栈帧等

栈帧： 每调用一个方法就有一个栈帧，比如main线程 main（）方法就是一个栈帧，在main 方法中调用了compute() ，就会为compute（）分配一个栈帧（栈帧把不同方法中的局部变量都隔离 开了） 。只会从栈顶往下一个一个执行完，从栈顶出栈

操作数栈：数字在做操作前临时保存的空间（比如int i =0, int j=1  ，这些0和1的数字都会先存到这里，如果有运算会运算结束后赋给局部变量表中）

动态链接：比如调用main.compute() ，这个compute就会保存在常量池中，在类加载的时候是不会解析的，只有程序运行的时候才会解析到对应的方法，方法中的变量会存到方法区中，会转换到方法区中的直接地址（后面细讲）

 方法出口：从main方法调用compute, 的这个位置，从main方法哪个位置出去的，要记录一下。一会compute执行完了之后要回到这个位置

局部变量表：放一些变量，或者引用 。比如Math math=new Math()  这个math就是引用（对应堆中的内存地址），而new Math() 会放到堆中。



## 本地方法栈

native一些c++ 写的一些方法，如果我们线程要调用这些方法的话，就会存到对应线程空间的本地方法栈中

## 方法区（元空间）

保存常量

+静态变量（比如static User user=new User ，这个静态user引用（内存地址）也会放到方法区，而new User就在堆中，引用指向堆）

+类元信息

##  程序计数器

程序的执行行数，每个线程都会有一个这个来记录当前程序执行到第几行了，也是在线程的空间中

字节码执行引擎在执行代码的过程中会不断修改这个数值

程序计数器主要是为了在多线程情况下保存当前执行的行数，当被其他线程抢过去之后回来要到这个行数下继续执行

## 堆

存对象的

栈、本地方法栈、程序计数器 这几个紫色的是每个线程独有的，而堆、方法区是共享的一块内存区



**补充一个问题：**

**在minor gc过程中对象挪动后，引用如何修改？**

对象在堆内部挪动的过程其实是复制，原有区域对象还在，一般不直接清理，JVM内部清理过程只是将对象分配指针移动到区域的头位置即可，比如扫描s0区域，扫到gcroot引用的非垃圾对象是将这些对象**复制**到s1或老年代，最后扫描完了将s0区域的对象分配指针移动到区域的起始位置即可，s0区域之前对象并不直接清理，当有新对象分配了，原有区域里的对象也就被清除了。

minor gc在根扫描过程中会记录所有被扫描到的对象引用(在年轻代这些引用很少，因为大部分都是垃圾对象不会扫描到)，如果引用的对象被复制到新地址了，最后会一并更新引用指向新地址。

这里面内部算法比较复杂，感兴趣可以参考R大的这篇文章：

https://www.zhihu.com/question/42181722/answer/145085437

https://hllvm-group.iteye.com/group/topic/39376#post-257329

# **二、JVM内存参数设置**

------

​    ![image-20220322203317844](https://image.imxyu.cn/file/image-20220322203317844.png)

Spring Boot程序的JVM参数设置格式(Tomcat启动直接加在bin目录下catalina.sh文件里)：

​                java -Xms2048M -Xmx2048M -Xmn1024M -Xss512K -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=256M -jar microservice-eureka-server.jar              

-Xss：每个线程的栈大小

-Xms：设置堆的初始可用大小，默认物理内存的1/64 

-Xmx：设置堆的最大可用大小，默认物理内存的1/4

-Xmn：新生代大小

-XX:NewRatio：默认2表示新生代占年老代的1/2，占整个堆内存的1/3。

-XX:SurvivorRatio：默认8表示一个survivor区占用1/8的Eden内存，即1/10的新生代内存。

关于元空间的JVM参数有两个：-XX:MetaspaceSize=N和 -XX:MaxMetaspaceSize=N

**-XX：MaxMetaspaceSize**： 设置元空间最大值， 默认是-1， 即不限制， 或者说只受限于本地内存大小。

**-XX：MetaspaceSize**： 指定元空间触发Fullgc的初始阈值(元空间无固定初始大小)， 以字节为单位，默认是21M左右，**达到该值就会触发full gc进行类型卸载**， 同时收集器会对该值进行调整： 如果释放了大量的空间， 就适当降低该值； 如果释放了很少的空间， 那么在不超过-XX：MaxMetaspaceSize（如果设置了的话） 的情况下， 适当提高该值。这个跟早期jdk版本的**-XX:PermSize**参数意思不一样，-**XX:PermSize**代表永久代的初始容量。

由于调整元空间的大小需要Full GC，这是非常昂贵的操作，如果应用在启动的时候发生大量Full GC，通常都是由于永久代或元空间发生了大小调整，基于这种情况，一般建议在JVM参数中将MetaspaceSize和MaxMetaspaceSize设置成一样的值，并设置得比初始值要大，对于8G物理内存的机器来说，一般我会将这两个值都设置为256M。

> 比如我们的程序启动如果元空间的MetaspaceSize初始内存设置的比较小，可能就会启动很慢，会加载大量对象（类元信息会放到这里），导致一直full gc ，去扩容。 

**StackOverflowError**示例：

       ```java
// JVM设置  -Xss128k(默认1M)
public class StackOverflowTest {
    
    static int count = 0;
    
    static void redo() {
        count++;
        redo();
    }
    
    public static void main(String[] args) {
        try {
            redo();
        } catch (Throwable t) {
            t.printStackTrace();
            System.out.println(count);
        }
    }
}

运行结果：
java.lang.StackOverflowError
	at com.tuling.jvm.StackOverflowTest.redo(StackOverflowTest.java:12)
	at com.tuling.jvm.StackOverflowTest.redo(StackOverflowTest.java:13)
	at com.tuling.jvm.StackOverflowTest.redo(StackOverflowTest.java:13)
   ......
       ```

​      

**结论：**

**-Xss设置越小count值越小，说明一个线程栈里能分配的栈帧就越少，但是对JVM整体来说能开启的线程数会更多**

**JVM内存参数大小该如何设置？**

JVM参数大小设置并没有固定标准，需要根据实际项目情况分析，给大家举个例子

# **日均百万级订单交易系统如何设置JVM参数**

  ![亿级流量电商系统JVM参数设置优化](https://image.imxyu.cn/file/亿级流量电商系统JVM参数设置优化.png)

1秒后方法运行完了，在方法中new 的这些对象就没有gc root 了，就变成垃圾对象了，等待minor垃圾回收器的回收。 而当14秒过后伊甸园区放满了，会进行minor的回收，此时会调用stop the world ,所有线程停止。但是此时方法还没有完成，也就是说对于当前的这60M的对象是有gcRoot 的，所以会把这时刻的60M对象放到survior 中，但是因为**对象动态年龄判断** ，60M大于survior 100M的百分50，所以不会放到survior，会直接放到old 老年代中。

此时老年代就会越来多，最后发生full gc

如何解决？ 发生问题的原因就是因为有对象动态年龄判断的机制，大于百分50了就直接放到old 区，所以我们需要将survior中的内存调大一点，或者直接将整个伊甸园区和新生代survior的内存调大，保证8:1:1 。这样的话会在伊甸园区和survior中转换， 下一次minor gc的时候会把这两个地方都回收，就不会到old了 

将old 中的2个G 移入一点到survior即可

**结论：**通过上面这些内容介绍，大家应该对JVM优化有些概念了，就是尽可能让对象都在新生代里分配和回收，尽量别让太多对象频繁进入老年代，避免频繁对老年代进行垃圾回收，同时给系统充足的内存大小，避免新生代频繁的进行垃圾回收。

**文档：02-VIP-JVM内存模型深度剖析与优化**

​                http://note.youdao.com/noteshare?id=ad3d29fc27ff8bd44e9a2448d3e2706d&sub=AC12369487BB46F2B3006BB4F3148D01              