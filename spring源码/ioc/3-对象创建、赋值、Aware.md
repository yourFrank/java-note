---

title: 3、Aware流程
tags:
  - spring源码
categories:
  - spring
  - spring源码
cover: 'https://image.imxyu.cn/file/image-20211207193924210.png'
description: Aware、@Autowired流程
abbrlink: a0b8b788
date: 2021-12-7 21:28:43
---



## 核心接口ApplicaitonContext

 ![image-20211205153759992](https://image.imxyu.cn/file/image-20211205153759992.png)

我们使用的**ApplicationContext接口**拥有如下功能

* ioc事件派发器
* 国际化解析
* bean工厂功能-- 自动装配是被组合进来的
* 资源解析功能



## Aware接口

如果我们想在自己写的类中使用这个ioc容器**ApplicationContext**  有下面两种方式：

1、通过自动注入

 ```java
  @Component
   public class Person  {
   
      @Autowired
      ApplicationContext context;  //可以要到ioc容器
      MessageSource messageSource;
       ...
   }
 ```

2、实现相应的Aware接口

这几个接口都是Aware接口，实现相应的接口后 在ioc容器 创建完Bean后都会将ioc容器（**ApplicationContext**）通过我们写的实现函数（回调）传入

![image-20211205154343395](https://image.imxyu.cn/file/image-20211205154343395.png)

```java
/**
 * Aware接口；帮我们装配Spring底层的一些组件
 * 1、Bean的功能增强全部都是有 BeanPostProcessor+InitializingBean  （合起来完成的）
 * 2、骚操作就是 BeanPostProcessor+InitializingBean
 *
 *
 */
public class Person implements ApplicationContextAware,MessageSourceAware{
    
	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		//利用回调机制，把ioc容器传入
		this.context = applicationContext;

	}
    
    @Override
	public void setMessageSource(MessageSource messageSource) {
		this.messageSource = messageSource;
	}

}
```

## debug位置

## 对象创建的过程

我们打两个断点看看何时创建Bean对象和 XXX Aware是什么时候把applicationContext赋值进来的

![image-20211205155259145](https://image.imxyu.cn/file/image-20211205155259145.png)



```java
/**
 * 因为使用的是注解，我们使用注解测试类
 */
public class AnnotationMainTest {

   public static void main(String[] args) {
      ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
      Person bean = applicationContext.getBean(Person.class);
      ApplicationContext context = bean.getContext();

      System.out.println(context == applicationContext);
   }
```

### 普通Bean对象创建的流程

容器创建的XML解析是和之前一样的

首先从注解类进入

![image-20211218193919514](https://image.imxyu.cn/file/image-20211218193919514.png)

容器刷新

![image-20211218193937015](https://image.imxyu.cn/file/image-20211218193937015.png)

我们看到不管是我们使用的AnnotationConfigApplicationContext 还是ClassPathXmlApplicationContext最终都是使用DefaultListableBeanFactory这个档案馆作为ioc容器。创建完成后也加载保存了所有的Bean定义信息(这个我们之前看了)

这次断点进入了完成BeanFactory初始化方法，此时工厂里的所有组件都好了，工厂可以正常工作了

![image-20211218194318124](https://image.imxyu.cn/file/image-20211218194318124.png)

在完成BeanFactory的初始化中预先初始化非懒加载的单实例Bean

![image-20211218194423777](https://image.imxyu.cn/file/image-20211218194423777.png)

首先获取所有的Bean定义信息，这个List就是我们之前注册的所有Bean定义信息，通过if判断我们可以看到将所有的**单实例、非抽象的、**

**非懒加载**，**并且不是工厂Bean的**挨个调用getBean（）（后面我们会看一下如果是工厂Bean，就会走上面的if）

![image-20211218195359884](https://image.imxyu.cn/file/image-20211218195359884.png)

![image-20211218195420991](https://image.imxyu.cn/file/image-20211218195420991.png)

在DefaultListableBeanFactory中调用**getBean（）**，这里使用了模板方法模式，getBean在父类中定义实现好了，子类调用。

> 模板方法：父类定义好了所有方法，父类实现了一部分，子类要么调用父类实现的方法，要么实现父类定义没有实现的方法

其实调用的父类（AbstractBeanFactory）中的getBean()

![image-20211218194857380](https://image.imxyu.cn/file/image-20211218194857380.png)

![image-20211218194842551](https://image.imxyu.cn/file/image-20211218194842551.png)

在doGetBean（）中，首先会检查缓存中有没有。

![image-20211218200031204](https://image.imxyu.cn/file/image-20211218200031204.png)



![image-20211218200114562](https://image.imxyu.cn/file/image-20211218200114562.png)





我们一直点进去，在**DefaultSingletonBeanRegistry**发现这是个Map，根据名字存储了所有的单例对象，**这就是单例对象池**，这个单例对象池就是我们之前说的[享元模式](https://imxyu.cn/javadoc/#/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/structural_type/%E7%BB%93%E6%9E%84%E5%9E%8B-%E7%BB%84%E5%90%88&%E5%A4%96%E8%A7%82&%E4%BA%AB%E5%85%83)(如果经常用的对象，我们可以放在池子中缓存，用的时候直接拿)。我们之前说过**DefaultListableBeanFactory**是存储Bean定义信息的，而这个类是存储所有创建好的单实例对象的池，并且这个类是DefaultListableBeanFactory的父类，我们一直使用的就是它的子类**DefaultListableBeanFactory**类这样既有了Bean定义信息又有单例对象池，这其实也是个模板（在父类中定义好了，子类直接使用)

![image-20211218200144472](https://image.imxyu.cn/file/image-20211218200144472.png)

![image-20211219090308833](https://image.imxyu.cn/file/image-20211219090308833.png)

第一次缓存中都是没有的，开始向下走。首先会看这个对象有没有depends on 依赖的对象，有的话先循环创建依赖对象。然后继续创建对象，这里使用lambda表达式，直接创建一个工厂对象，通过工厂对象创建实例

![image-20211218200317993](https://image.imxyu.cn/file/image-20211218200317993.png)

这个工厂对象是一个函数式接口

![image-20211218200934265](https://image.imxyu.cn/file/image-20211218200934265.png)

调用的工厂的getObject（）方法，我们使用lambda创建工厂的时候重写了getObject（），调用createBean（）方法

![image-20211218201028131](https://image.imxyu.cn/file/image-20211218201028131.png)

我们重写了getObject( )，调用createBean( )，这里进入了createBean（）。createBean这个方法也是使用了模板方法模式，父类中定义了，子类在这里实现的

![image-20211218203310000](https://image.imxyu.cn/file/image-20211218203310000.png)

在doCreateBean这个方法中调用创建Bean对象

![image-20211218201259654](https://image.imxyu.cn/file/image-20211218201259654.png)

进去之后，首先策略模式获取相应的创建对象的策略，点进这个获取策略的方法看看

![image-20211218201416565](https://image.imxyu.cn/file/image-20211218201416565.png)

这个工厂就是这个策略模式的环境类，组合了策略接口

![image-20211218201508373](https://image.imxyu.cn/file/image-20211218201508373.png)

这个策略接口有两个实现，一个是简单策略，一个是通过cglib代理的策略

![image-20211218201535418](https://image.imxyu.cn/file/image-20211218201535418.png)

这里选用了简单策略创建，直接调用了BeanUtils（spring自己的工具类）

![image-20211218202049224](https://image.imxyu.cn/file/image-20211218202049224.png)

调用反射创建对象

![image-20211218202142769](https://image.imxyu.cn/file/image-20211218202142769.png)

调用了我们写的构造函数

![image-20211218202225091](https://image.imxyu.cn/file/image-20211218202225091.png)

---

**至此对象创建就已经完成了。**

**接下来我们看一下aware赋值是什么时候，我们直接放行到下一个断点：**

## ApplicationContextAwareProcessor回调Aware接口赋值

![image-20211218202822407](https://image.imxyu.cn/file/image-20211218202822407.png)

我们从这个调用lambda表达式创建工厂继续看

![image-20211218202836437](https://image.imxyu.cn/file/image-20211218202836437.png)

进入我们重写的createBean

![image-20211218203310000](C:\Users\tianyu\AppData\Roaming\Typora\typora-user-images\image-20211218203310000.png)

这次在创建完Bean对象之后，断点进入了一个初始化Bean的方法

![image-20211218203728125](https://image.imxyu.cn/file/image-20211218203728125.png)

执行后置处理器

![image-20211218204109043](https://image.imxyu.cn/file/image-20211218204109043.png)

获取所有的BeanPostProcessor类型的后置处理器执行

![image-20211218204130912](https://image.imxyu.cn/file/image-20211218204130912.png)

因为我们实现了ApplicationContextAware 和MessageAware，所以进入下面的else逻辑。

这个接口是我们后置处理器**BeanProcessor**中的一个，所有的功能增强都是通过后置处理器完成的。如果我们想创建完Bean后进行功能增强，都是通过这些后置处理器完成的，后面我们会详细讲一下这些后置处理器

![image-20211218205327435](https://image.imxyu.cn/file/image-20211218205327435.png)

![image-20211218204419602](https://image.imxyu.cn/file/image-20211218204419602.png)



进行判断，直接回调我们写的set方法，把applicationContext帮我们传入。

![image-20211218204513892](https://image.imxyu.cn/file/image-20211218204513892.png)

> 在这里我们也可以看到，不管我们实现了Aware的这任意的接口，都把applicationContext或者它的属性给我们传进来了。这也很合理，因为之前说过applicationContext继承了国际化、事件派发、环境这些接口

![image-20211218204531509](https://image.imxyu.cn/file/image-20211218204531509.png)

上面就是通过实现Aware接口，spring通过底层回调我们的方法来帮我们赋值的过程

## xml属性赋值的过程

### debug位置

在Person的setName处打断点，看何时属性赋值

![image-20211207203959724](https://image.imxyu.cn/file/image-20211207203959724.png)

```java
public class MainTest {

	public static void main(String[] args) {
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
		Person bean = context.getBean(Person.class);
		System.out.println(bean);
	}

}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

	<bean class="cn.imlql.spring.bean.Person" id="person">
		<property name="name" value="张三"/>
 	</bean>

</beans>
```



![image-20211219100840086](https://image.imxyu.cn/file/image-20211219100840086.png)

### 流程

进入doCreatebean，在这里我们能看到之前对象创建和Aware进入的位置

![image-20211219101401378](https://image.imxyu.cn/file/image-20211219101401378.png)

进入属性赋值方法，首先拿到了Bean定义信息中所有的键值对（xml文件中写的那些），我们看一下这个PropertyValues。

![image-20211219102038660](https://image.imxyu.cn/file/image-20211219102038660.png)

这个PropertyValues使用了**迭代器模式**，保存了属性的键值对。PropertyValues的key对应的就是bean的属性，values对应了属性的值![image-20211219102109990](https://image.imxyu.cn/file/image-20211219102109990.png)

也就是我们xml中定义的，之前创建工厂的时候已经保存到档案馆中的。

![image-20211219103035119](https://image.imxyu.cn/file/image-20211219103035119.png)

这里进入

![image-20211219102555388](https://image.imxyu.cn/file/image-20211219102555388.png)

将这个所有对应的属性深拷贝一份，然后对person对象赋值

![image-20211219103157884](https://image.imxyu.cn/file/image-20211219103157884.png)

继续调用

![image-20211219103322798](https://image.imxyu.cn/file/image-20211219103322798.png)

遍历所有的属性赋值

![image-20211219103404948](https://image.imxyu.cn/file/image-20211219103404948.png)

继续调用赋值

![image-20211219103507657](https://image.imxyu.cn/file/image-20211219103507657.png)

![image-20211219103554650](https://image.imxyu.cn/file/image-20211219103554650.png)

利用反射调用set方法，getWapperedInstance（）就是包装后的Person对象，value就是张三

![image-20211219103639954](https://image.imxyu.cn/file/image-20211219103639954.png)

然后就到了我们的构造函数

![image-20211219103726087](https://image.imxyu.cn/file/image-20211219103726087.png)



## @Autowired属性注入

刚刚我们看了在xml文件中定义的是如何赋值的，下面我们看看使用@Autowired注解属性注入的是如何赋值的

### debug位置

Person类：

因为这里直接给属性赋值，不太好debug，这里给方法@Autowired注入看

![image-20211207213027784](https://image.imxyu.cn/file/image-20211207213027784.png)

```java
//使用注解类debug @Autowired
public class AnnotationMainTest {
   public static void main(String[] args) {

      ApplicationContext applicationContext =
            new AnnotationConfigApplicationContext(MainConfig.class);

      Person bean = applicationContext.getBean(Person.class);
      ApplicationContext context = bean.getContext();
      System.out.println(context == applicationContext);
   }
}
```

### 流程

![image-20211219105734444](https://image.imxyu.cn/file/image-20211219105734444.png)

进入之后这里可以看到我们我们之前讲的xml赋值的，在这里获取Bean自动赋值的后置处理器实行注解的注入

![image-20211219105943797](https://image.imxyu.cn/file/image-20211219105943797.png)

获取所有实现了**InstantiationAwareBeanPostProcessor**后置处理器（spring的自动注入注解就是通过实现了这个后置处理器接口来实现的）实行注入(这里又是一个后置处理器)，并且传入所有Bean属性键值对pvs，已经创建好的Person，和beanName

![image-20211219110311575](https://image.imxyu.cn/file/image-20211219110311575.png)

![image-20211219110534946](https://image.imxyu.cn/file/image-20211219110534946.png)

进入之后先找到所有标注自动装配注解的元信息，我们点进这个方法看一下

![image-20211219110942778](https://image.imxyu.cn/file/image-20211219110942778.png)

获取到的所有包含注解的信息，并且包装成InjectionMetadata返回

![image-20211219111327785](https://image.imxyu.cn/file/image-20211219111327785.png)

1、找所有属性中标注了@Autowired\@Value\@Inject注解 （使用反射工具类）

2、拿到所有方法，看有没有@Autowired注解

找到之后添加到elements 集合中，之后再包装成InjectionMetadata类型返回（这里又用到了装饰器）

![image-20211219111546051](https://image.imxyu.cn/file/image-20211219111546051.png)

找标记了下面注解的属性

![image-20211219111606086](https://image.imxyu.cn/file/image-20211219111606086.png)

然后我们继续回来，找到所有注解标注的属性和方法之后，调用inject注入

![image-20211219112431839](https://image.imxyu.cn/file/image-20211219112431839.png)

这里找到了两个，一个是在属性标的@Autowired，一个在方法中标记的@Autowired

![image-20211219112421400](https://image.imxyu.cn/file/image-20211219112421400.png)

在inject中调用method方法反射赋值

![image-20211219112636040](https://image.imxyu.cn/file/image-20211219112636040.png)

进入到我们的set构造函数

![image-20211219112828414](https://image.imxyu.cn/file/image-20211219112828414.png)



## 流程图



![image-20211207193924210 ](https://image.imxyu.cn/file/image-20211207193924210.png)

