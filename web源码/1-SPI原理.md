## SPI的定义

要理解spring mvc 原理，首先要理解SPI

SPI的启发初始来源于serviceLoader ,是java写的一个规范

我们来看一个demo

```java
public class MainTest {

    public static void main(String[] args) {

        //1、加载 可用的接口实现
        ServiceLoader<DataSaveService> load = ServiceLoader.load(DataSaveService.class);

        //拿到实现进行调用
        for (DataSaveService service : load) {
            service.saveData("你好....");
        }

    }
}
```

ServiceLoader 有一个load方法，参数可以传入一个接口。

调用load() 方法后，就会加载当前系统里面所有的这个接口的【实现】

接下来我们来演示一下

首先我们创建了一个接口模块

![image-20220208202105757](https://image.imxyu.cn/file/image-20220208202105757.png)

这个模块可以用来数据保存服务的接口，我们单独抽成了一个模块

接下来，我们创建又分别创建了两个模块，并且创建了相应的实现类去实现这个接口，做相应的数据保存服务



![image-20220208202236298](https://image.imxyu.cn/file/image-20220208202236298.png)

```java
public class RedisSaveService implements DataSaveService {
    @Override
    public void saveData(String data) {
        System.out.println("Redis保存了数据......."+data);
    }
}
```

```java
public class MySQLSaveService implements DataSaveService
{
    @Override
    public void saveData(String data) {
        System.out.println("MySQL保存了数据......."+data);
    }
}

```

这两个模块分别在pom.xml 中都引入这个接口的坐标，所以可以引用到接口类

```xml
   <dependencies>
        <dependency>
            <groupId>com.atguigu</groupId>
            <artifactId>api-db-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>


```



## SPI的使用

### 实现类中创建文件

```
* SPI（Service Provider Interface）
*      接口工程---提供接口
*          ---- 实现工程1  ： 实现接口 【META-INF/services 创建文件  接口名作为文件名  实现类全路径作为文件内容】
*          ---- 实现工程2  ： 实现接口
*
*
*      客户端----引用 工程1、或者 工程2
*
```

使用SPI，我们可以让客户端引用工程1就能得到工程1的实现，引用工程2就能得到2的实现，两个都引用了就都能得到相应的实现

那么我们如何做到？

需要让工程1（接口的实现类）【类路径下 META-INF/services 创建文件  接口名作为文件名  实现类全路径作为文件内容】

 ![image-20220208203043818](https://image.imxyu.cn/file/image-20220208203043818.png)

文件的内容为实现类的全路径（这里以redis的为例，就是redisService实现类的全路径）

```
com.atguigu.redis.RedisSaveService
```

同样的对于mysql工程的实现类也是同样的操作

![image-20220208203222627](https://image.imxyu.cn/file/image-20220208203222627.png)

### 客户端中引用实现类所在的工程

![image-20220208203440132](https://image.imxyu.cn/file/image-20220208203440132.png)

> 这里因为这两个工程都引入了接口，这里不引入接口也是可以的，依赖传递会过来

### 客户端中使用ServiceLoader指定接口得到实现

这样就可以进行使用了

```java
public class MainTest {

    public static void main(String[] args) {

        //1、加载 可用的接口实现
        ServiceLoader<DataSaveService> load = ServiceLoader.load(DataSaveService.class);

        //拿到实现进行调用
        for (DataSaveService service : load) {
            service.saveData("你好....");
        }

    }
}

```

因为ServiceLoader实现了interable接口，是可迭代的。所以我们可以for循环就得到我们引入的所有实现

![image-20220208203638666](https://image.imxyu.cn/file/image-20220208203638666.png)



运行结果：

![image-20220208203819561](https://image.imxyu.cn/file/image-20220208203819561.png)

因为我们引入了两个这两个工程（实现类），所以for循环得到了这两个实现

> 如果我们取消一个依赖引入，就会只有一个。 两个都取消就没有，这里就不演示了



## SPI的好处

SPI 的好处就是以后我们只用定义接口，开放给任何人实现，我们可以拿到任意的接口实现来用。这就是java 实现的一套规范

## 总结

总结下来就是

实现类实现接口，并且创建文件指定实现的哪个接口，文件内容写明实现类的全路径

客户端引用相应的实现类

客户端中使用ServiceLoader 指定接口，然后就能得到引用的实现（引用了几个就能得到几个）

## SPI应用

 Dubbo和Spring MVC+Tomcat的整合

明白了SPI，以后看这两个的源码就轻松多了