## 父子容器

我们先来回顾一下以前web.xml中的配置

![image-20220219135656319](https://image.imxyu.cn/file/image-20220219135656319.png)

spring和springMvc 产生了父子容器

在FrameworkServlet中设置了父子容器

因此我们可以在controller 中获取到service， 就是因为首先会在子容器中找，找不到会去父容器中找

![image-20220219135832697](https://image.imxyu.cn/file/image-20220219135832697.png)



## 更快的整合springMvc

### AbstractContextLoaderInitializer利用servlet监听器机制初始化webioc容器



![image-20220219140158075](https://image.imxyu.cn/file/image-20220219140158075.png)

```
继承AbstractAnnotationConfigDispatcherServletInitializer
```

之前我们说过根据SPI机制项目启动的时候会去加载WebApplicationInializer 的实现

调用WebApplicationInializer 各个实现类的onStartup()方法

```
AbstractContextLoaderInitializer这个类的
```

![image-20220219140634947](https://image.imxyu.cn/file/image-20220219140634947.png)

这个onStartup 方法中，调用registerContextLoaderListener（）注册这个监听器

而ServletContextListener监听器也是一个servlet，遵循servlet规范

这个contextLoaderListener继承了它

![image-20220219141222182](https://image.imxyu.cn/file/image-20220219141222182.png)

之前说过web启动之后，对于servlet会调用他们的初始化方法，因此这个监听器会在当前web应用启动以后（Tomcat把web应用加载了以后），调用contextInitialized初始化方法创建了webioc容器

![image-20220219141342820](https://image.imxyu.cn/file/image-20220219141342820.png)



### AbstractAnnotationConfigDispatcherServletInitializer这个子类中实现了初始化了父ioc容器（root）

我们再回到刚才的registerContextLoaderListener 中

在第一步中

WebApplicationContext rootAppContext = createRootApplicationContext(); //创建一个根容器

这个我们还没看，我们点进去看看

![image-20220219141808548](https://image.imxyu.cn/file/image-20220219141808548.png)



我们点进来发现，这个方法是抽象的，AbstractContextLoaderInitializer留给了子类去实现这个方法

![image-20220219141845895](https://image.imxyu.cn/file/image-20220219141845895.png)

我们看一下子类中对它的实现，看它的子类

```java
AbstractAnnotationConfigDispatcherServletInitializer  //这个类实现了这个方法
```

![image-20220219142000997](https://image.imxyu.cn/file/image-20220219142000997.png)

在这个类中重写了这个方法

**首先获取根配置类，然后创建ioc容器并把配置类注册进来**

![image-20220219142031660](https://image.imxyu.cn/file/image-20220219142031660.png)

我们发现getRootConfigClass（） 还是抽象的，需要留给子类去实现的模板方法

而下面这里还有一个getServletConfigClasses（） ，获取web的配置类，我们看一下是在哪里用到的这个

![image-20220219142330291](https://image.imxyu.cn/file/image-20220219142330291.png)

发现也是在这个类中，而这个方法是什么时候被调用的呢？

点进去一看发现这里创建了root ioc容器，并且需要web容器的配置类注册

![image-20220219142610155](https://image.imxyu.cn/file/image-20220219142610155.png)

发现是在这个类初始化的时候（因为这个类也是webApplicationInitailizer的实现子类），因此onStart方法会被调用

1、 首先调用super.onStartup() .之前我们说过在父类中创建了webioc 容器，并且将rootioc容器（父容器）的方法抽象。而这个子类实现了这个抽象方法，创建了父容器AnnotationConfigWebApplicationContext（其中留了一个抽象方法给我们实现去指定root ioc容器的配置类）

2、获取子类中web容器的配置类（需要我们实现去指定的），将web容器的配置类传入



![image-20220219142826586](https://image.imxyu.cn/file/image-20220219142826586.png)



>  说了那么多，因此我们就不需要之前的appStarter去自己实现WebApplicationInitializer，我们只需要继承AbstractAnnotationConfigDispatcherServletInitializer这个类并且指定好配置类就可以完成父子容器的初始化了

```java
/**
 * 最快速的整合注解版SpringMVC和Spring的
 */
public class QuickAppStarter extends AbstractAnnotationConfigDispatcherServletInitializer {
	@Override //根容器的配置（Spring的配置文件===Spring的配置类）
	protected Class<?>[] getRootConfigClasses() {
		return new Class<?>[]{SpringConfig.class};
	}

	@Override //web容器的配置（SpringMVC的配置文件===SpringMVC的配置类）
	protected Class<?>[] getServletConfigClasses() {
		return new Class<?>[]{SpringMVCConfig.class};
	}

	@Override //Servlet的映射，DispatcherServlet的映射路径
	protected String[] getServletMappings() {
		return new String[]{"/"};
	}

	@Override
	protected void customizeRegistration(ServletRegistration.Dynamic registration) {
//		super.customizeRegistration(registration);

//		registration.addMapping("");//
	}
}

```

并且创建两个配置类

```java
/**
 * Spring不扫描controller组件、AOP咋实现的....
 */
@ComponentScan(value = "com.atguigu.web",excludeFilters = {
		@ComponentScan.Filter(type= FilterType.ANNOTATION,value = Controller.class)
})
@Configuration
public class SpringConfig {
	//Spring的父容器

}

```

```java
/**
 * SpringMVC只扫描controller组件，可以不指定父容器类，让MVC扫所有。@Component+@RequestMapping就生效了
 */
@ComponentScan(value = "com.atguigu.web",includeFilters = {
		@ComponentScan.Filter(type= FilterType.ANNOTATION,value = Controller.class)
},useDefaultFilters = false)
public class SpringMVCConfig {
	//之前说过子容器中设置了父容器
    //SpringMVC的子容器，能扫描的Spring容器中的组件，而spring容器中扫描不到web中的，这样单向引用就比较安全


}

```

## 父子容器的启动过程

我们给这个类打上断点，看一下父子容器的启动过程

![image-20220219150435882](https://image.imxyu.cn/file/image-20220219150435882.png)

### 第一个ioc容器创建

1、 首先根据SPI机制加载到这个SpringServletContainerInitializer类

在这个类中获取了所有的实现了WebApplicationInitializer接口的类，其中就有我们实现的

![image-20220219151018322](https://image.imxyu.cn/file/image-20220219151018322.png)

然后for循环调用所有的onStartup()方法，我们看一下我们的QuickStart中是如何调用的

![image-20220219151314886](https://image.imxyu.cn/file/image-20220219151314886.png)

而我们的onStartup（）是继承的父类的，首先会调用父类的onStartup()

我们点进去看看

![image-20220219151412727](https://image.imxyu.cn/file/image-20220219151412727.png)

```
调用registerContextLoaderListener()
```

![image-20220219151501799](https://image.imxyu.cn/file/image-20220219151501799.png)

这个我们上面看过了，就是在爷爷类中调用的创建RootApplicationContext

只不过这个方法没有实现，在孙子类中对它进行了实现

![image-20220219151724927](https://image.imxyu.cn/file/image-20220219151724927.png)

```java
AbstractAnnotationConfigDispatcherServletInitializer //类中首先会获取根配置，然后创建一个父容器
```

![image-20220219151828457](https://image.imxyu.cn/file/image-20220219151828457.png)

这样就调到了我们实现的根容器配置文件位置的重写方法

![image-20220219151929130](https://image.imxyu.cn/file/image-20220219151929130.png)

获取完配置类之后创建了容器，并且把我们的配置类添加了进去  

**此时容器还没有刷新，是一个空容器**

### 监听器保存了spring容器

2、创建完了spring容器，在父类这个方法中调用 new ContextLoaderListener(rootAppContext);

**创建了一个监听器，并且将我们的刚刚创建的ioc容器保存了进去**

此时这个registerContextLoaderListener（）方法就结束了

![image-20220219153148591](https://image.imxyu.cn/file/image-20220219153148591.png)

```java
super.onStartup(servletContext); //父类的方法调用也就完成了
```

而这个监听器会在tomcat容器启动之后调用contextInitialized方法（到时候我们再看）

我们在这里留一个断点，等到tomcat启动完成之后会调用到这里完成容器的初始化

![image-20220219155207536](https://image.imxyu.cn/file/image-20220219155207536.png)

### 第二个ioc容器创建

我们接下来看看这个registerDispatcherServlet方法

![image-20220219153535065](https://image.imxyu.cn/file/image-20220219153535065.png)

```
WebApplicationContext servletAppContext = createServletApplicationContext(); //创建Servlet容器，我们点进去看看
```

![image-20220219153807897](https://image.imxyu.cn/file/image-20220219153807897.png)

发现这个里面又创建了一个容器，这个就是web容器，然后调用getServletConfigClasses() 方法，获取web容器的配置类

![image-20220219153835485](https://image.imxyu.cn/file/image-20220219153835485.png)

此时就到了我们写的这个方法

![image-20220219153921949](https://image.imxyu.cn/file/image-20220219153921949.png)

### dispatcherServlet保存了web-ioc容器

接下来创建了dispatcherServlet ，保存了这个ioc容器

![image-20220219154612167](https://image.imxyu.cn/file/image-20220219154612167.png)

```
ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet); //然后将我们new 好的servlet添加进去
```

```
registration.addMapping(getServletMappings()); //根据我们指定的DispatcherServlet的路径进行注册
```

这个方法也是抽象的留给我们子类实现的，因此也到了我们的实现当中

![image-20220219154521046](https://image.imxyu.cn/file/image-20220219154521046.png)

然后如果有filter，会添加一些filter

```
customizeRegistration(registration); //这里又留了一个模板方法留给我们实现的
```

我们也可以实现这个方法，去对servlet里面进行额外的操作，比如说添加映射等

![image-20220219154835265](https://image.imxyu.cn/file/image-20220219154835265.png)

此时两个容器就创建完成了，代码部分就结束了

### tomcat 回调监听器/servlet初始化方法

#### 回调监听器init->根容器刷新

tomcat容器完成之后就会回调servlet的容器初始化流程

首先来到上面监听器的初始化方法（之前说过监听器也是实现了servlet规范）

来到了ContextLoaderListener 这个监听器，这里的参数是我们上面说的设置的根容器

![image-20220219155309462](https://image.imxyu.cn/file/image-20220219155309462.png)

这里面继续调用configureAndRefreshWebApplicationContext

![image-20220219155734596](https://image.imxyu.cn/file/image-20220219155734596.png)



![image-20220219155837842](https://image.imxyu.cn/file/image-20220219155837842.png)

来到了我们容器刷新的十二大步

![image-20220219155855638](https://image.imxyu.cn/file/image-20220219155855638.png)

我们放行

此时 HelloService 创建，因为我们的根容器配置的时候只扫描除了web的，所以这里controller没有创建

![image-20220219161109135](https://image.imxyu.cn/file/image-20220219161109135.png)

至此根容器的刷新就完成了

![image-20220219202615096](https://image.imxyu.cn/file/image-20220219202615096.png)

#### 回调servlet的init->web容器刷新

继续对其他servlet对象进行初始化方法

我们继续放行，来到了HttpServletBean，这个servlet

**因为我们上面创建了DispatcherServlet，并且保存了web容器，所以调用了DispatcherServlet的初始化方法	**

public abstract class HttpServletBean extends HttpServlet implements EnvironmentCapable, EnvironmentAware {

![image-20220219202717716](https://image.imxyu.cn/file/image-20220219202717716.png)我们看一下这个留给子类的模板方法模式initServletBean（)

![image-20220219203000929](https://image.imxyu.cn/file/image-20220219203000929.png)

点进去这个初始化webIOC容器的方法

**可以看到这个初始化web容器调用的方法也是上面根容器的调用方法initWebApplicationContext（）**

在这个方法中首先调用，获取到servletContext中已经有的容器

```
WebApplicationContext rootContext =
      WebApplicationContextUtils.getWebApplicationContext(getServletContext()); //父容器
```

![image-20220219203027245](https://image.imxyu.cn/file/image-20220219203027245.png)

可以看到这个方法就是我们之前熟悉的servlet的作用域，上面根容器创建好了之后被保存到了applicationContext作用域中，因此这里获取到了根容器

![image-20220219203224369](https://image.imxyu.cn/file/image-20220219203224369.png)

```java
WebApplicationContext rootContext   //这个就是得到的父容器已经初始化好的spring容器
this.webApplicationContext  //而这个就是我们当前的web-ioc容器
```

![image-20220219203550235](https://image.imxyu.cn/file/image-20220219203550235.png)

通过调用cwac.setParent(rootContext); 组合了父子容器

接下来调用这个方法刷新子容器

```
configureAndRefreshWebApplicationContext(cwac); //配置并且刷新容器
```

同样在web容器刷新的过程中，前面设置一些属性，然后就调用refresh（）方法，走进了容器刷新的十二大步

![image-20220219204123423](https://image.imxyu.cn/file/image-20220219204123423.png)

接下来就是web-ioc 容器的刷新过程

![image-20220219204355418](https://image.imxyu.cn/file/image-20220219204355418.png)

继续放行，就来到了controller 的创建

也就是说web容器中只扫描了controller的创建，并且在controller的对象创建过程中需要自动装配service，就会在父容器中找。

> 也就是说只能单向装配，从controller 中装配service，不能反着来会报错
>
> 原因就是先刷新的根容器，然后刷新的web-ioc容器

![image-20220219204439621](https://image.imxyu.cn/file/image-20220219204439621.png)

在web容器中使用parent字段保存了spring容器

![image-20220219204734407](https://image.imxyu.cn/file/image-20220219204734407.png)

## 父子容器启动流程图

![image-20220219205804791](https://image.imxyu.cn/file/image-20220219205804791.png)

![SpringMVC原理](https://image.imxyu.cn/file/SpringMVC%E5%8E%9F%E7%90%86.jpg)

