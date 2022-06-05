# AOP

## 方法论

### 基本点

* 看文档注释、方法注释
* 翻译方法名、变量名（自解释模式）
* 注意类结构；

### spring套路点

* AbstractBeanDefinition 看看何时给容器中注入了什么组件
* BeanFactory 让初始化完，监控里面多了哪些后置处理器
* 分析后置处理器什么时候调用

> 以上所有的前提，理解容器刷新12大步与getBean 流程，放置混乱
>
> 1、工厂后置处理器执行
>
> 2、bean后置处理器执行&bean的生命周期（后置处理器+initializingBean）

1、这个新功能一般都是由**bean生命周期机制**增强出来的

2、这个功能加入了哪些组件，这些组件在生命周期中做了什么？

## 引入依赖

使用gradle

```groovy
compile(project(":spring-aspects"))  //引入aop&切面模块
```



## 编写测试

 配置类

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@EnableAspectJAutoProxy //开启自动代理
@Configuration
public class AopOpenConfig {


}

```

另一个配置类是我们之前的，用来包扫描的，当扫描到上面的配置类就会加入进去

![image-20220108150633666](https://image.imxyu.cn/file/image-20220108150633666.png)

切面类

```java
@Component  //切面也是容器中的组件
@Aspect //说明这是切面
public class LogAspect {

	public LogAspect(){
		System.out.println("LogAspect...");
	}

	//前置通知  增强方法/增强器
	@Before("execution(* com.atguigu.spring.aop.HelloService.sayHello(..))")
	public void logStart(JoinPoint joinPoint){
		String name = joinPoint.getSignature().getName();
		System.out.println("logStart()==>"+name+"....【args: "+ Arrays.asList(joinPoint.getArgs()) +"】");
	}

	//返回通知
	@AfterReturning(value = "execution(* com.atguigu.spring.aop.HelloService.sayHello(..))",returning = "result")
	public void logReturn(JoinPoint joinPoint,Object result){
		String name = joinPoint.getSignature().getName();
		System.out.println("logReturn()==>"+name+"....【args: "+ Arrays.asList(joinPoint.getArgs()) +"】【result: "+result+"】");
	}


	//后置通知
	@After("execution(* com.atguigu.spring.aop.HelloService.sayHello(..))")
	public void logEnd(JoinPoint joinPoint){
		String name = joinPoint.getSignature().getName();
		System.out.println("logEnd()==>"+name+"....【args: "+ Arrays.asList(joinPoint.getArgs()) +"】");
	}


	//异常
	@AfterThrowing(value = "execution(* com.atguigu.spring.aop.HelloService.sayHello(..))",throwing = "e")
	public void logError(JoinPoint joinPoint,Exception e){
		String name = joinPoint.getSignature().getName();
		System.out.println("logError()==>"+name+"....【args: "+ Arrays.asList(joinPoint.getArgs()) +"】【exception: "+e+"】");
	}
}
```

业务类

```java
@Component  //切面存在的化就会返回代理对象
public class HelloService {

	public HelloService(){
		System.out.println("....");
	}

	public String sayHello(String name){
		String result = "你好："+name;
		System.out.println(result);
		int length = name.length();
		return result + "---" + length;
	}
}

```

切面里面的所有通知方法会在目标方法的前后进行回调

直接运行,调用我们的

![image-20220108150843473](https://image.imxyu.cn/file/image-20220108150843473.png)

## 原理

1、AOP给容器中添加了什么组件

>  1、AbstractBeanDefinition  的构造器打断点，就能知道容器中的bean定义信息，spring 也可能在底层直接new对象注册进去，最好在refresh()十二大步的最后一步打上断点，在debug控制台看有哪些没见过的组件。单独分析他们即可
>
> 2、每一个功能的开启，要么写配置，要么注解（springboot就是这样的，每个功能的开启都是注解@Enablexxx）。@EnableXXX开启xxx功能的注解。这个注解很重要

比如说我们这里

```java
@EnableAspectJAutoProxy //开启自动代理
@Configuration
public class AopOpenConfig {

}
```

@EnableAspectJAutoProxy 点进去看看

之前说过给容器中添加组件可以用@Import

@Import(AspectJAutoProxyRegistrar.class)

这个注解中的类可以实现ImportBeanDefinitionRegistrar接口，可以自己在BeanDefinition注册组件



![image-20220108154138450](https://image.imxyu.cn/file/image-20220108154138450.png)

比如说这个类就是实现了这个接口，可以自己实现组件的注册

我们给这个里打个断点，看一下

![image-20220108160444047](https://image.imxyu.cn/file/image-20220108160444047.png)

### 1、AOP注解中的@Import注册该后置处理器定义信息

#### 后置处理器解析配置类的时候发现了@Import注解

我们通过堆栈，找到进入的入口

在十二大步中工厂增强中进入，在这里会执行所有的工厂的后置处理器。而配置类的后置处理器也在这里执行

![image-20220108160639120](https://image.imxyu.cn/file/image-20220108160639120.png)

![image-20220108160752670](https://image.imxyu.cn/file/image-20220108160752670.png)

找到配置类的后置处理器，执行后置处理器的方法

![image-20220108160824804](https://image.imxyu.cn/file/image-20220108160824804.png)

这里面会有处理配置类的Bean定义信息

![image-20220108160925266](https://image.imxyu.cn/file/image-20220108160925266.png)

这里后置处理器如何知道哪个是配置类呢？这里就是得到所有的bean定义信息然后找到配置类

![image-20220108161223936](https://image.imxyu.cn/file/image-20220108161223936.png)

下面就是解析配置类,通过reader加载配置类中的bean定义信息

![image-20220108161409005](https://image.imxyu.cn/file/image-20220108161409005.png)

处理的过程中发现了@Import注解

![image-20220108161543671](https://image.imxyu.cn/file/image-20220108161543671.png)

#### 通过接口回调注册aop后置处理器的定义信息

方法调用来注册bean定义信息

![image-20220108161710051](https://image.imxyu.cn/file/image-20220108161710051.png)

我们的类实现了这个接口。直接这个类调用这个方法就回调到了我们的方法中

![image-20220108162224668](https://image.imxyu.cn/file/image-20220108162224668.png)

调到这个注解类实现的接口方法

![image-20220108161831174](https://image.imxyu.cn/file/image-20220108161831174.png)

我们继续跟进去看看，它给容器中注册了个什么组件？一直跟

![image-20220108162431009](https://image.imxyu.cn/file/image-20220108162431009.png)

 传入了一个AnnotationAwareAspectJAutoProxyCreator类型的Class

![image-20220108163000700](https://image.imxyu.cn/file/image-20220108163000700.png)

继续调用首先会看一下容器中是否有这个类型的组件，第一次肯定是没有的

然后就会new了一个AnnotationAwareAspectJAutoProxyCreator这个类型的组件的定义信息，加入到容器中

![image-20220108163151630](https://image.imxyu.cn/file/image-20220108163151630.png)

我们看一下这个类，点开他的类图

![image-20220108163400107](https://image.imxyu.cn/file/image-20220108163400107.png)

可以看到这个组件和它的父类是一个后置处理器

![image-20220108163549099](https://image.imxyu.cn/file/image-20220108163549099.png)

AOP就是用这个后置处理器来帮我们执行功能的



> 我们分析其他的功能也是，就看看这个注解给我们的容器中加了什么后置处理器，这个后置处理器做了什么功能

### 2、这个后置处理器做了什么功能？

#### debug位置

首先我们可以看继承图看出来这是一个BeanPostProcessor，那么我们如何知道这个组件做了什么活？

1、首先这是BeanPostProcessor，执行是在bean的生命周期函数中，我们可以追踪getBean的**整个流程**，看什么时候AnnotationAwareAspectJAutoProxyCreator执行（最笨的办法）

2、我们可以直接找到这个类，给关键的方法打断点

此时在容器中导入了 

AnnotationAwareAspectJAutoProxyCreator这个类

为了看这个组件做了什么功能，我们给它的重写方法打断点。

> 因为重写方法是从父类继承重写的，其他的非@Override方法是一些get/set 值的，如果我们从重写的方法中没有获得有效的信息，再去考虑其他的方法

![image-20220109103103629](https://image.imxyu.cn/file/image-20220109103103629.png)

因此我们给这个类的几个重写的方法打上断点，看一下

![image-20220109103312758](https://image.imxyu.cn/file/image-20220109103312758.png)

#### aop后置处理器的创建和初始化

我们debug运行，首先来到了initBeanFactory。我们追踪栈来看一下

![image-20220109103740783](https://image.imxyu.cn/file/image-20220109103740783.png)

可以看到refresh（） 12大步，这是从注册Bean的后置处理器这里进来的

我们之前分析过，这里是注册所有的**BeanPostProcessor**，创建**Bean**后置处理器的对象添加到容器中

![image-20220109103828625](https://image.imxyu.cn/file/image-20220109103828625.png)

继续调用，这个PostProcessorRegistrationDelegate我们之前说过。

所有的Bean的后置处理器和BeanFactory的后置处理器都是通过这个类来调用的

![image-20220109104003515](https://image.imxyu.cn/file/image-20220109104003515.png)

可以看出，这个后置处理器是实现了Ordered接口的，在这里获取到了所有Ordered接口的Bean后置处理器的定义信息

然后调用Getbean创建对象

![image-20220109104404906](https://image.imxyu.cn/file/image-20220109104404906.png)

接下来就是调用getBean，lambda，doCreateBean，createBean那一些来创造对象

![image-20220109104621583](https://image.imxyu.cn/file/image-20220109104621583.png)

![image-20220109104632725](https://image.imxyu.cn/file/image-20220109104632725.png)

![image-20220109104728254](https://image.imxyu.cn/file/image-20220109104728254.png)

这里对象创建好了之后，进入了初始化bean的里面

![image-20220109104811004](https://image.imxyu.cn/file/image-20220109104811004.png)

为什么会进初始化呢？我们再看一下这个类的继承图，发现它还实现了Aware接口

所以这里会把BeanFactory给它回调传进去

![image-20220109105045076](https://image.imxyu.cn/file/image-20220109105045076.png)

在这里调用

![image-20220109105418782](https://image.imxyu.cn/file/image-20220109105418782.png)

因为实现了BeanFactoryAware接口

所以将当前bean转成BeanFactoryAware类型，调用setBeanFactory（）把AbstractAutowireCapableBeanFactory传入

![image-20220109105425660](https://image.imxyu.cn/file/image-20220109105425660.png)

这里子类继续重写了这个方法，调用的是子类重写的，

![image-20220109105611091](https://image.imxyu.cn/file/image-20220109105611091.png)

然后就来到了这里，可以看出来这个是从BeanFactoryAware接口调用来的

这里获取到了beanFactory工厂

![image-20220109105825382](https://image.imxyu.cn/file/image-20220109105825382.png)

调用父类的super,然后将beanFactory 保存到了advisorRetrievalHelper属性里

![image-20220109111726407](https://image.imxyu.cn/file/image-20220109111726407.png)

然后下面创建了两个对象，有临时属性保存了起来

他们就是我们后置增强器功能的两个对象。我们直接放行到下一个断点

![image-20220109112056756](https://image.imxyu.cn/file/image-20220109112056756.png)

#### 切面类的判断

断点来到了这里，看名字是判断是否是基础设施的类，我们看一下

就是在判断切面类，我们看一下在什么时候判断切面类

![image-20220109112907569](https://image.imxyu.cn/file/image-20220109112907569.png)

判断的时候看是否实现了下面的几个接口

![image-20220109112949123](https://image.imxyu.cn/file/image-20220109112949123.png)

或者是标有@Aspect注解的

![image-20220109113140420](https://image.imxyu.cn/file/image-20220109113140420.png)

我们看一下是什么时候进行这个判断的？追踪栈

#### 其他组件创建对象的时候aop后置处理器开始增强

发现还是从这里进来的，继续点

![image-20220109113247003](https://image.imxyu.cn/file/image-20220109113247003.png)

发现是mybeanPostProcessor，这里既不是我们的切面也不是aop相关的东西的时候触发了

这里调用getBean的时候触发了

![image-20220109113511787](https://image.imxyu.cn/file/image-20220109113511787.png)

下面就是getBean的一系列流程，就不截图了。发现是在这里调用的

原因是：aop的后置处理器的对象已经创建好了（因为它是实现了ordered的接口的后置处理器，所以比其他普通的后置处理器创建的要早），而当普通的后置处理器在创建的时候aop的后置处理器就开始它的增强逻辑，并且后面所有的bean对象在创建进行bean的生命周期的时候，aop的后置处理器都会在这里做增强逻辑

![image-20220110190408709](https://image.imxyu.cn/file/image-20220110190408709.png)

执行bean的相关的后置处理器

![image-20220110190802152](https://image.imxyu.cn/file/image-20220110190802152.png)

获取所有InstantiationAwareBeanPostProcessor类型的后置处理器，这里就找到了aop的后置处理器

![image-20220110190939814](https://image.imxyu.cn/file/image-20220110190939814.png)

这里会判断，是否是切面类或者是否需要跳过

![image-20220110192355504](https://image.imxyu.cn/file/image-20220110192355504.png)

这里判断isInfrastructureClass，就是调用的我们这个方法。于是就来到了我们的断点这里

![image-20220110192444746](https://image.imxyu.cn/file/image-20220110192444746.png)

#### 第一次就找到切面类和增强方法保存起来

我们再看一下它的另一个判断shouldSkip

List<Advisor> candidateAdvisors = findCandidateAdvisors();  在这个方法中首先找到候选的增强器，我们来看一下它是如何找到的

![image-20220110192614011](https://image.imxyu.cn/file/image-20220110192614011.png)

发现这个方法是我们的AnnotationAwareAspectJAutoProxyCreator后置处理器中中打了断点的方法

在这个方法中首先找到它的所有增强器，这里因为要创建的对象是myBeanPostProcessor，不是我们的切面类也不是我们要增强的类。

此时如果有aspectJAdvisorsBuilder不为空（这个是上面初始化BeanFactory的时候new出来的）

this.aspectJAdvisorsBuilder.buildAspectJAdvisors() ；这个方法要构建增强器

我们看一下这个方法是如果构建的增强器，点进去看看

![image-20220110193239773](https://image.imxyu.cn/file/image-20220110193239773.png)

第一次进来，这个切面名字为空，也就是说此时还不知道哪个是切面，我们来看一下如果为空是怎么处理的

```java
String[] beanNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
      this.beanFactory, Object.class, true, false);
      
```

发现这里直接获取了所有类型为object的所有bean的名字

![image-20220110193455555](https://image.imxyu.cn/file/image-20220110193455555.png)

这里找到了24个，也就是所有的beanName了

![image-20220110193828396](https://image.imxyu.cn/file/image-20220110193828396.png)

下面循环遍历所有的beanname，然后判断是否是切面，如果是的话放入到集合当中

然后获取切面的增强器

![image-20220110194119211](https://image.imxyu.cn/file/image-20220110194119211.png)

其中判断切面很简单，就是看类里面有没有Aspect注解

![image-20220110194314574](https://image.imxyu.cn/file/image-20220110194314574.png)

![image-20220110194327353](https://image.imxyu.cn/file/image-20220110194327353.png)

可以看到这里找到了切面是logAspect的类，然后调用getAdvisors（factory）找到切面类中的增强器

![image-20220110194236382](https://image.imxyu.cn/file/image-20220110194236382.png)

这里找到了切面类中的所有增强方法，然后保存了起来

![image-20220110194529397](https://image.imxyu.cn/file/image-20220110194529397.png)

找到了之后将他们都保存起来了

![image-20220110194659382](https://image.imxyu.cn/file/image-20220110194659382.png)



我们继续放行到下一个断点

发现下一次来到了我们的切面类的判断

我们看一下是从哪里进来的

![image-20220110194845787](https://image.imxyu.cn/file/image-20220110194845787.png)

发现还是这个myBeanPostProcessor, 第一次是在创建对象的时候进来的，这一次是在创建完对象赋值后的初始化Bean触发了aop的后置处理器

![image-20220110194923556](https://image.imxyu.cn/file/image-20220110194923556.png)

![image-20220110195151514](https://image.imxyu.cn/file/image-20220110195151514.png)

判断当前对象是否需要返回一个包装对象

这里会判断，如果是要被切入的对象，要返回它相应的包装对象，这里就是在判断

![image-20220110195242721](https://image.imxyu.cn/file/image-20220110195242721.png)

判断是否是切面类，或者应该被跳过

![image-20220110195413175](https://image.imxyu.cn/file/image-20220110195413175.png)

此时又到这里的时候，aspectBeanNames在第一次就已经找到了切面保存了起来，所以这时候就已经有值了

aspectBeanNames 保存了所有的切面类的名，classAdvisors保存了该切面对应的增强器

advisorsCache这个Map保存了切面类和其增强器（也就是那四个方法）

> 可以看到spring经常用双重检查锁的形式，先判断是否为null,然后加锁，又一次判断。我们写代码的时候也可以使用这种，来提高安全性和效率

![image-20220110195530383](https://image.imxyu.cn/file/image-20220110195530383.png)



也就是说，还没有等到我们用我们的HelloService（被切入的）类的时候，spring就已经提前帮我们找到了切面类（LogAspect）和所有的切面方法，并保存了起来

#### 如何获取的所有增强器

这里说的增强器，其实就是增强方法，Advisor

我们再来细看一下这一步是如何找到的所有增强方法的

![image-20220110201547079](https://image.imxyu.cn/file/image-20220110201547079.png)

首先得到我们的切面类，这里直接得到了Class类型的类对象

然后下面getAdvisorMethods（）方法会获取该类的所有方法，我们点进去看看

![image-20220110202157644](https://image.imxyu.cn/file/image-20220110202157644.png)

这里通过反射工具类，其中doWithMethods方法的参数有一个函数式接口methods::add 

![image-20220110202307665](https://image.imxyu.cn/file/image-20220110202307665.png)

这里mc是一个回调，这里直接通过方法引用来将getDeclaredMethods反射获取的所有方法全部加入到mehods集合中

![image-20220110202407726](https://image.imxyu.cn/file/image-20220110202407726.png)

通过这个找到了所有的方法，接下来就是遍历这15个方法（其中有我们写的增强方法，也有从object类中继承下来的方法）

![image-20220110202555794](https://image.imxyu.cn/file/image-20220110202555794.png)

我们点进去下面的getAdvisor（）方法，看是如何获取其中的增强器的

第一个方法是logAsepct方法，这是我们定义的增强方法

它首先会调用getPointcut（）方法来获取pointcut注解

![image-20220110202952543](https://image.imxyu.cn/file/image-20220110202952543.png)

下面就是反射获取这几个类型的注解

![image-20220110203056569](https://image.imxyu.cn/file/image-20220110203056569.png)

![image-20220110203105424](https://image.imxyu.cn/file/image-20220110203105424.png)

通过判断之后发现这个是增强器，就加入到advisors集合当中

![image-20220110203226194](https://image.imxyu.cn/file/image-20220110203226194.png)

就这样15个方法，就被我们筛选过后，找到了这4个增强的方法并且添加到了集合当中

![image-20220110203336903](https://image.imxyu.cn/file/image-20220110203336903.png)

下面还会获取所有需要增强的属性，我们没有写

可以看到是通过getDeclareParentsAdvisor方法去获取的

![image-20220110203628371](https://image.imxyu.cn/file/image-20220110203628371.png)

主要是找这个DeclareParents类型的注解

![image-20220110203733765](https://image.imxyu.cn/file/image-20220110203733765.png)

这样就找到了4个增强器，然后会进行判断，看当前切面类是否是单例的，如果是单例的就放入advisorsCache中

否则就直接放入aspectFactoryCache 

![image-20220110203812251](https://image.imxyu.cn/file/image-20220110203812251.png)

