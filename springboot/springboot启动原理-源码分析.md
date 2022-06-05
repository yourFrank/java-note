

## 项目搭建

springboot的源码其实就是基于spring和springMVC的功能基础上给我们自动配置了一些包

接下来我们看一下springboot的源码，首先构建一个简单的springboot工程,这里只建了一个web的工程

点开这个web的依赖看一下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.4</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.atuigu.boot</groupId>
    <artifactId>springboot-source</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-source</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

其实这个web依赖给我们引入了很多包，这里面也包含了我们之前自己写过的tomcat包

![image-20220305191342323](https://image.imxyu.cn/file/image-20220305191342323.png)

点开tomcat包，里面依赖了这些，这就是我们之前自己实现tomcat引入的包，这里springboot帮我们引入了

![image-20220305191424947](https://image.imxyu.cn/file/image-20220305191424947.png)



一个简单的springboot只需要这两个注解和这个main方法就可以启动起来，第一个注解@Configuration就不用说了，声明这是一个配置类，之前我们说过spring源码，当spring容器启动的时候，会有一个后置处理器ConfigurationClassPostProcessor 会帮们分析所有的配置类，去加载配置类中的组件

我们主要来看一下@EnableAutoConfiguration 这个注解

![image-20220305191150931](https://image.imxyu.cn/file/image-20220305191150931.png)

## @EnableAutoConfiguration注解

这个注解包含了这两个

![image-20220305191736781](https://image.imxyu.cn/file/image-20220305191736781.png)

### @AutoConfigurationPackage

其中这个注解主要是导入了这个类

![image-20220305191850901](https://image.imxyu.cn/file/image-20220305191850901.png)

我们来可以看到这个类实现了ImportBeanDefinitionRegistrar接口，之前我们说过，实现了这个接口可以拿到registry，可以自己向容器中注入组件

接下来我们看一下这个类给我们注入了什么组件

![image-20220305192038929](https://image.imxyu.cn/file/image-20220305192038929.png)

首先会拿到注解的元信息，就是用来描述注解的注解的信息，也就是拿到了标在最外层的@EnableAutoConfiguration 的注解的信息，得到了所在的类名还有类上的注解，

然后获取到了包名，传到register中

```java
new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames()
```

![image-20220305192714035](https://image.imxyu.cn/file/image-20220305192714035.png)



首先拿到了我们的包名，basePackageName, 然后new了一个bean定义信息，然后加入到容器中。这样就指定了我们以后要扫描的这个包下的组件

![image-20220305192222986](https://image.imxyu.cn/file/image-20220305192222986.png)

beanname是这个

![image-20220305193535247](https://image.imxyu.cn/file/image-20220305193535247.png)



这个执行完了之后就有了这个bean定义信息，当处理到这个组件的时候（会标明这是一个包处理的定义信息）然后就会帮我们导入这个包下的组件，到时候我们再来观察看创建这个bean的时候是如何帮我们扫描包，导入包下的所有组件的

![image-20220305193858295](https://image.imxyu.cn/file/image-20220305193858295.png)

接下来我们看一下，下一个注解是做了什么的



### @Import({AutoConfigurationImportSelector.class})

点开这个导入的类之后，里面有一个重写的process() 方法，主要也是这个方法帮我们导入组件的，我们看一下

![image-20220305194410370](https://image.imxyu.cn/file/image-20220305194410370.png)

首先也是会拿到元注解的信息，这里拿到了元注解类上的所有注解的信息

接下来这句是拿到所有自动配置的集合

```java
AutoConfigurationEntry autoConfigurationEntry = ((AutoConfigurationImportSelector) deferredImportSelector)
      .getAutoConfigurationEntry(annotationMetadata);
```

拿到所有的集合后都会put放入entries，我们进去看一下getAutoConfigurationEntry

![image-20220305194720534](https://image.imxyu.cn/file/image-20220305194720534.png)

通过这句要拿到候选的配置信息

```
List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
```

![image-20220305195001260](https://image.imxyu.cn/file/image-20220305195001260.png)



通过工厂名字加载

```java
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
```

![image-20220305195032031](https://image.imxyu.cn/file/image-20220305195032031.png)

加载的就是这个类，我们点进去看一下是怎么加载的

![image-20220305195158947](https://image.imxyu.cn/file/image-20220305195158947.png)

![image-20220305195258552](https://image.imxyu.cn/file/image-20220305195258552.png)

在加载的时候会去找这个文件

> 这也是spring自己定义的SPI机制

![image-20220305195356992](https://image.imxyu.cn/file/image-20220305195356992.png)

拿到这个文件后，接下来去找文件中的这个值

注意找这个文件也是在当前**类路径**下找的

![image-20220305195808071](https://image.imxyu.cn/file/image-20220305195808071.png)

![image-20220305195738486](https://image.imxyu.cn/file/image-20220305195738486.png)

在类路径下的这么多jar包中，在自动配置包中找到了这个文件

![image-20220305195911408](https://image.imxyu.cn/file/image-20220305195911408.png)

主要去根据找这个key所对应的值的

![image-20220305195957146](https://image.imxyu.cn/file/image-20220305195957146.png)



通过这个方法找到了130个相应的值，我们随便进去一个类看一下

![image-20220305200233099](https://image.imxyu.cn/file/image-20220305200233099.png)

比如这个自动配置类，也是一个@configuration

![image-20220305200305192](https://image.imxyu.cn/file/image-20220305200305192.png)

这些所有的**配置类**，注意说到配置类都只有一个功能，就是通过@Bean来给容器中放东西

然后就进入这些bean的生命周期等等。。。。就是spring的事情了

![image-20220305200418398](https://image.imxyu.cn/file/image-20220305200418398.png)

接下来会将这些组件进行一些自己指定的排除

![image-20220305200840080](https://image.imxyu.cn/file/image-20220305200840080.png)

可以自定义排除一些组件，我们这里没有定义。所以还是130个组件

![image-20220305200935309](https://image.imxyu.cn/file/image-20220305200935309.png)

接下来会通过过滤的工作，这个过滤也就是说这130个组件不会全部生效，我们随便点开一个看看

![image-20220305201035878](https://image.imxyu.cn/file/image-20220305201035878.png)

比方说jdbc的，这些类上都有@ConditionalOnClass，这个注解后面会判断如果我们导入了这些类才会生效

所以这样过滤完了之后就只剩下了23个，也就是说我们只有根据不同的场景导入不同的jar包，这些自动配置类才会生效被加入到容器中帮我们注入一些组件

![image-20220305201144619](https://image.imxyu.cn/file/image-20220305201144619.png)



接下来我们就可以分析这些生效的自动配置组件，看给我们注入了什么组件

比方说我们的disptcherServlet自动配置类

![image-20220305201426315](https://image.imxyu.cn/file/image-20220305201426315.png)

这里面就会给我们添加了很多组件， 比方说disptatcherServlet还有文件上传的组件等等。下面我们就来分析

![image-20220305201630289](https://image.imxyu.cn/file/image-20220305201630289.png)



## 容器刷新onRefresh()启动tomcat

首先我们来介绍一个注解

```java
@EnableConfigurationProperties(WebMvcProperties.class)
```

通过这个注解可以让绑定配置文件

![image-20220305210901551](https://image.imxyu.cn/file/image-20220305210901551.png)

也就是说我们可以在springboot的application.properties 配置文件中写这个前缀的，到时候属性就会加载到这个类中

然后上面会得到这个类的对象，然后通过我们放置的属性进行初始化组件的值

![image-20220305210947721](https://image.imxyu.cn/file/image-20220305210947721.png)



接下来我们看一下这个注解，也就是说在DispatcherServletAutoConfiguration 进行自动配置要在这个注解中设置的类之后才会执行

![image-20220305211105359](https://image.imxyu.cn/file/image-20220305211105359.png)

接下来我们看这个类，上面的这两个注解

![image-20220305211240684](https://image.imxyu.cn/file/image-20220305211240684.png)

第一个注解表示的是服务器的配置文件，也就是说我们可以写server.port= 9888 , 来指定服务器的启动端口等

![image-20220305211342737](https://image.imxyu.cn/file/image-20220305211342737.png)

第二个注解为我们导入了这4个类，首先我们来看看第一个

这个是一个bean后置处理器

```
 ServletWebServerFactoryAutoConfiguration.BeanPostProcessorsRegistrar.class,
```

![image-20220305211554684](https://image.imxyu.cn/file/image-20220305211554684.png)



后面这三个组件都是服务器相关的，第一个是tomcat，2 jetty,3 undertow 。这三个组件会根据我们导入了相应的包，会选择任意的组件，可以看到这里我们有tomcat相关的依赖类

并且这三个类上都有@ConditionalOnMissingBean(value = ServletWebServerFactory.class, search = SearchStrategy.CURRENT)

**也就是说我们可以自己往容器中加入自定义的服务器，如果没有的话才会选择加入这三个中的**

![image-20220305211632185](https://image.imxyu.cn/file/image-20220305211632185.png)

![image-20220305211638089](https://image.imxyu.cn/file/image-20220305211638089.png)



这是服务器工厂的接口，我们可以自己实现，然后注册到容器中，这样就会用我们的服务器，（比方说我想用apache等等）。



![image-20220305211929955](https://image.imxyu.cn/file/image-20220305211929955.png)

![image-20220305211947770](https://image.imxyu.cn/file/image-20220305211947770.png)

>  我们说tomcat中放置dispatcherServlet可以利用SPI机制，同时我们也可以直接new一个dispatchServlet放入到容器中

### 在自动配置注册bean定义信息的时候创建web服务器工厂

我们说了dispatcherServletAutoConfigure 会在servletWebServerFactoryConfiguration 完成后再进行，

那么我们看一下这个自动配置类servletWebServerFactoryConfiguration做了什么，是怎么创建web服务器的

因此我们这里用的是springboot默认的tomcat，我们来看一下这里是怎么创建的tomcat服务器

> 注意@Bean里面只是向里面添加bean定义信息的过程，此时的bean还没有创建呢

![image-20220305213728285](https://image.imxyu.cn/file/image-20220305213728285.png)

@Bean方法中这里面的参数是从容器中拿的，也就是说我们可以自己定义这些组件放到容器中，然后就可以在方法中注入我们自己的组件

![image-20220305213930442](https://image.imxyu.cn/file/image-20220305213930442.png)



第一步是拿到我们的web服务器工厂，既然是服务器工厂，我们上面看到过，服务器工厂接口的方法有getWebServer（） 这个方法获取web服务器

```
TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
```

> 注意这里只是创建tomcat的web服务器工厂，是用来创建tomcat的，tomcat创建还不在这里
>
> 因为是web服务器工厂，我们看一下它的getWebServer（），就是创建tomcat的方法，看一下是什么时候调用的？

我们找到这个类中的getWebServer方法

这里面就是非常熟悉的代码，new Tomcat() ，这些我们之前也自己写过的代码

我们直接给这里打一个断点，看一下是什么时候调用了这个getWebServer方法

![image-20220305214505899](https://image.imxyu.cn/file/image-20220305214505899.png)

### springboot启动后在容器onRefresh（）的时候用到这个工厂，调用启动tomcat

我们来追踪栈

首先springBoot启动

![image-20220305214718852](https://image.imxyu.cn/file/image-20220305214718852.png)

接下来可以看到我们创建了一个注解类型的容器

![image-20220305214906719](https://image.imxyu.cn/file/image-20220305214906719.png)

然后容器刷新，在刷新的十二大步的onRefresh方法中

之前我们一直没有用到这个方法

![image-20220305214707480](https://image.imxyu.cn/file/image-20220305214707480.png)

可以看到是这个子类实现了这个方法

![image-20220305215024786](https://image.imxyu.cn/file/image-20220305215024786.png)

实际我们用的是注解的那个子类，它的父类定义了这个方法，而模板方法在Abstract中写的

![image-20220305215156494](https://image.imxyu.cn/file/image-20220305215156494.png)

### 调用getWebServer方法

在容器刷新 createWebServer的时候调用了getWebServer

![image-20220305215043719](https://image.imxyu.cn/file/image-20220305215043719.png)



![image-20220305215637390](https://image.imxyu.cn/file/image-20220305215637390.png)



### getWebServer中的prePareContext()方法

我们先点进去看一下这个方法

首先会把默认的servlet和jsp添加进去

![image-20220306112001992](https://image.imxyu.cn/file/image-20220306112001992.png)

这就是默认的能处理的请求路径

![image-20220306112039222](https://image.imxyu.cn/file/image-20220306112039222.png)

接下来会得到所有的servletContextInitializer类型的类

![image-20220306111011382](https://image.imxyu.cn/file/image-20220306111011382.png)

在这个方法中会new TomcatStarter starter = new TomcatStarter(initializers); 把所有的ServeltContextInitializer保存到tomcatStarter中

![image-20220306111243383](https://image.imxyu.cn/file/image-20220306111243383.png)

当tomcat .start() 启动后调用onStart up （）的时候，会调用所有ServeltContextInitializer的 onStartup 方法

![image-20220306111217345](https://image.imxyu.cn/file/image-20220306111217345.png)

这个方法会调用register(description, servletContext); 

![image-20220306111417197](https://image.imxyu.cn/file/image-20220306111417197.png)

![image-20220306111536413](https://image.imxyu.cn/file/image-20220306111536413.png)

把所有的ServeltContextInitializer 包括url Mapping 能处理的请求都放进去

![image-20220306111551288](https://image.imxyu.cn/file/image-20220306111551288.png)

这就是prepareContext 需要做的工作，

> 后面我们会说servlerContextInitializer类型 是干什么的，tomcat 为什么启动后要注册这个类型的组件

![image-20220306112316378](https://image.imxyu.cn/file/image-20220306112316378.png)

然后会调用这个方法

```
return getTomcatWebServer(tomcat);
```

![image-20220305215714388](https://image.imxyu.cn/file/image-20220305215714388.png)

![image-20220305215720297](https://image.imxyu.cn/file/image-20220305215720297.png)



在这个初始化方法中启动了tomcat

![image-20220305215747239](https://image.imxyu.cn/file/image-20220305215747239.png)



## tomcat启动加载DispatcherServelt时机

之前我们说过tomcat加载dispatcehrservlet有两种方法

一种是通过SPI机制，还有一种可以直接tomcat.addDispatcherServelt ，直接添加

而springboot就是通过addDispatcherServlet的方式将tomcat添加到容器中的

接下来我们看一下tomcat什么时候添加到容器的

我们首先来看一下dispatcherServletConfiguration这个自动配置类，上面说过这个类会在servletWebServerFactoryAutoConfiguration之后再执行，而这个servletWebServerFactoryAutoConfiguration我们上面看到了，就是准备了一个服务器工厂，等到容器刷新的时候会调用这个工厂创建出tomcat

我们再来看一下这个dispatcherServletConfiguration，首先会创建一个dispatcherServlet的定义信息添加到容器中

> 注意此时还没有用，因为只有放到tomcat中，才会在tomcat启动的时候调用servlet的初始化方法进行工作

![image-20220306105235402](https://image.imxyu.cn/file/image-20220306105235402.png)

接下来我们看一下下面还有一个注册组件的@Bean，这个首先拿到了我们上面的dispatcherServlet 注册的对象，

然后会转换成DispatcherServletRegistrationBean 这个对象，添加到容器中，我们看一下这个类型的bean是干什么的



![image-20220306105428541](https://image.imxyu.cn/file/image-20220306105428541.png)

发现这个类型属于serveltContextIniializer 

![image-20220306105701712](https://image.imxyu.cn/file/image-20220306105701712.png)

上面我们说过了，当onRefresh 的时候会去加载这个类型的组件放到tomcatServer中，然后启动tomcat的时候，会把所有的serveltContextIniializer 添加到tomcat 中，而我们的disptatcherServlet被保存到了DispatcherServletRegistrationBean 中，这个bean的类型就是serveltContextIniializer 

这样大家就明白了



## 在tomcat中注册原生的组件



比如说我们想注册原生的serlvet组件，添加自己能处理的的urlMappings 映射，我们就可以注册一个ServletRegistrationBean

然后把我们的servlet放到里面，tomcat就会通过上面我们说过的那样放入到tomcat中

![image-20220306112832519](https://image.imxyu.cn/file/image-20220306112832519.png)

![image-20220306113134442](https://image.imxyu.cn/file/image-20220306113134442.png)



比方说我们想注册一个原生的filter到tomcat 中

我们可以用同样的方法，通过注册FilterRegistrationBean到容器中，因为它底层就是servletContextInitializer

![image-20220306113322694](https://image.imxyu.cn/file/image-20220306113322694.png)

> 总结：
>
> ```
> //所有的xxxRegistrationBean都是允许我们注册原生的Servlet组件进去，
> //利用 ServletContextInitializer在Tomcat启动完成以后进行回调的机制
> 
> ```
>
> tomcat 在底层就会调用tomcat .addServlet 等帮我们注册进去

## springboot启动原理

我们再来看一下springboot的启动类

![image-20220306140713872](https://image.imxyu.cn/file/image-20220306140713872.png)



![image-20220306140727253](https://image.imxyu.cn/file/image-20220306140727253.png)



![image-20220306140734722](https://image.imxyu.cn/file/image-20220306140734722.png)

我们点进去刷新容器看一下

![image-20220306140801124](https://image.imxyu.cn/file/image-20220306140801124.png)



![image-20220306140825371](https://image.imxyu.cn/file/image-20220306140825371.png)

![image-20220306140832715](https://image.imxyu.cn/file/image-20220306140832715.png)

![image-20220306140838506](https://image.imxyu.cn/file/image-20220306140838506.png)

进入容器刷新的12大步，在OnRefresh 方法中会启动tomcat

而tomcat启动会加载tomcat中所有的servlet进入初始化方法, 这里面就有dispatcherServelt .然后进入dispatcherServlet初始化方法，加载九大组件

![image-20220306141033444](https://image.imxyu.cn/file/image-20220306141033444.png)



> 1、SpringBoot.run会创建一个ioc容器，
>
> AnnotationConfigServletWebServerApplicationContext2、ioc容器启动onRefresh会启动Tomcat
>
> 3、Tomcat启动会加载所有的Servlet
>
> 4、DispatcherServlet会加载九大组件的整个初始化流程

## 流程图

![SpringBoot原理](https://image.imxyu.cn/file/SpringBoot%E5%8E%9F%E7%90%86.jpg)



