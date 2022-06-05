

## Spring实现了servlet的规范接口

![image-20220209193308926](https://image.imxyu.cn/file/image-20220209193308926.png)

首先我们看AppStarter 实现了WebApplicationInitializer 接口，我们点进去看看

这个接口有一个方法，然后我们看它上面标注了一个实现SpringServletContainerInitializer，我们点进去

![image-20220209193354989](https://image.imxyu.cn/file/image-20220209193354989.png)

实现了这个接口ServletContainerInitializer，我们点进去这个接口

![image-20220209193502794](https://image.imxyu.cn/file/image-20220209193502794.png)

这个接口就是servlet 的规范，而我们的springMVC实现了这个规范

![image-20220209193630958](https://image.imxyu.cn/file/image-20220209193630958.png)

可以看到这个就是我们之前说的SPI规范

**META-INF/services 的文件名为ServletContainerInitializer这个接口名，而里面就是实现类的全路径SpringServletContainerInitializer**

也就是上面我们看到的这个接口的实现

**因此Tomcat一启动就会根据SPI规范，去加载SpringServletContainerInitializer**

![image-20220209193720857](https://image.imxyu.cn/file/image-20220209193720857.png)



## 调用了所有实现WebApplicationInitializer的类onStartup()

可以看到SpringServletContainerInitializer implements ServletContainerInitializer

而上面有一个@HandlesTypes 注解，这个注解里面是感兴趣的类（WebApplicationInitializer.class）



![image-20220209194449852](https://image.imxyu.cn/file/image-20220209194449852.png)

而我们的AppStarter恰好实现了WebApplicationInitializer 接口，

![image-20220209194943438](https://image.imxyu.cn/file/image-20220209194943438.png)



可以看到容器一启动所有实现了WebApplicationInitializer接口的就都被注入进来了![image-20220209194931409](https://image.imxyu.cn/file/image-20220209194931409.png)

接下来for循环所有的webAppInitializerClasses ,给他们反射创建对象，

然后for循环调用onStartup（）方法，这样就调到了我们AppStarter实现的onStartup（）方法

![image-20220209200810221](https://image.imxyu.cn/file/image-20220209200810221.png)

这样断点来到了AppStarter的onStartup方法

### ioc容器创建（空容器）

这个方法中我们首先创建了ioc容器，在创建ioc容器的过程中并没有启动刷新容器

然后创建了DispatcherServlet，容器刷新的过程是在这里进行的？

![image-20220209201330957](https://image.imxyu.cn/file/image-20220209201330957.png)

### servlet创建启动->ioc容器刷新

因为DispatcherServlet是servlet

而每个servlet接口有它的生命周期，Tomcat在启动的时候会为没一个servlet都创建一个单例对象

Servlet规范会经历它的生命周期：

1、Servlet创建对象

2、Servlet调用init初始化

3、每次请求过来调用service方法处理

4、tomcat停止营业会调用destory销毁

因此下面的子类中必须要提供init（）方法

![image-20220209201559862](https://image.imxyu.cn/file/image-20220209201559862.png)

### HttpServletBean实现了init初始化方法

在这个子类中实现了这个init方法

![image-20220209202133463](https://image.imxyu.cn/file/image-20220209202133463.png)

并且留了一个模板方法initServletBean();

![image-20220209202209136](https://image.imxyu.cn/file/image-20220209202209136.png)

### FrameworkServlet在HttpServletBean留的模板方法中初始化ioc容器

这个模板方法在子类FrameworkServlet中实现了。

在这里我们看到了一句关键的话

```java
this.webApplicationContext = initWebApplicationContext(); //初始化WebIOC容器
```

spring容器就是在这里进行创建的

![image-20220209202326300](https://image.imxyu.cn/file/image-20220209202326300.png)



接下来就是走spring容器刷新的十二大步

## 原理图

> 右键在标签页中打开图片，ctrl 缩放浏览器查看大图

![SpringMVC原理(1)](https://image.imxyu.cn/file/SpringMVC%E5%8E%9F%E7%90%86(1).jpg)