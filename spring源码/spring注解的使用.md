---
title: Spring常用注解
tags:
  - spring
categories:
  - spring
  - 用法
keywords: Spring，框架
description: 一些常用的注解
cover: 'https://image.imxyu.cn/file/spring-avater01.webp'
abbrlink: 1a003b7b
date: 2021-10-24 22:21:58
---

**声明**：

1、学习思路全部来自B站尚硅谷雷丰阳老师的教学视频：https://b23.tv/I8QvhX

2、笔记来自https://www.yuque.com/niuweijiu/dhwvbv/gevcgu#5LsOm 鸡尾酒，自己简单排版删减部分

![spring_mind](https://image.imxyu.cn/file/spring_mind.jpg)


## 向容器中注入实例对象

###  XML文件方式

1. 在pom中添加spring-context依赖

```xml
<dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>4.3.12.RELEASE</version>
</dependency>
```

2. 定义一个实体类（此处使用lombok注解提供有参、无参构造器以及get,set方法）

```java
@Data 
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String name;
    private Integer age;
}
```

3. beans.xml 配置文件中通过<bean></bean> 标签注入类实例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" 
       xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
     http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context-4.0.xsd ">
    <!-- 注入实例对象 -->
    <bean id="person01" class="com.example.bean.Person">
        <property name="name" value="bob"/>
        <property name="age" value="18"/>
    </bean>
</beans>
```

4. 获取容器中通过配置文件注入的实例对象

```java
public class MainTest {
  
   @SuppressWarnings("resource")
	public static void main(String[] args) {
        // 读取通过配置文件，创建ioc容器，单例对象此时在容器创建的时候就已经注入对象
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
        // 根据bean的id获取容器中注入
		Person bean = (Person) applicationContext.getBean("person");
		System.out.println(bean);
	}
}
```

5. 结果的输出

```
Person [name=zhangsan, age=18, nickName=null]
```

### 注解方式 @Bean+@Configuration

1、2 步不变

3. 不再配置xml文件，而是新建一个配置类MyConfig.java（名字可以随意），在配置类文件中通过注解向容器中注入实例对象。

```java
//配置类==配置文件，这就相当于之前的配置文件
@Configuration  //告诉Spring这是一个配置类
public class MainConfig {
   
   //给容器中注册一个Bean;类型为返回值的类型，id默认是用方法名作为id(就是bean的名字)，在这里就是person01
   @Bean
   public Person person01(){
      return new Person("lisi", 20);
   }
  // 如果需要自定义id的值，只需在@Bean()的参数部分设置就行
    @Bean(person01)
    public Person person(){
        return new Person("lisi",20);
    }

}
```

4. 获取容器中通过配置类注入的实例对象

```java
public class Main {
    public static void main(String[] args) {
        // 这里因为使用注解的方式注册的，此时我们需要new AnnotationConfigApplicationContext 容器
        ApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig.class);
        Person person1 = (Person) annotationConfigApplicationContext.getBean("person");
        System.out.println(person1);
    }
}
```

### 包扫描

#### XML配置包扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" 
       xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
     http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context-4.0.xsd ">

    <!-- 配置包扫描，use-default-filters不配置的话默认过滤规则为false,该包下所有包含@Controller/@Service/@Component/Repository的任意注解都会加入进来 -->
    <context:component-scan base-package="com.example" use-default-filters="false" />
    <!-- 注入实例对象 -->
    <bean id="person01" class="com.example.bean.Person">
        <property name="name" value="bob"/>
        <property name="age" value="18"/>
    </bean>
</beans>
```

####  注解方式配置包扫描

```java
//配置类==配置文件
@Configuration  //告诉Spring这是一个配置类

@ComponentScans(
      value = {
            @ComponentScan(value="com.atguigu",includeFilters = {
/*                @Filter(type=FilterType.ANNOTATION,classes={Controller.class}),
                  @Filter(type=FilterType.ASSIGNABLE_TYPE,classes={BookService.class}),*/
                  @Filter(type=FilterType.CUSTOM,classes={MyTypeFilter.class})//自定义过滤规则
            },useDefaultFilters = false)   
      }
      )
//@ComponentScan  value:指定要扫描的包
//excludeFilters = Filter[] ：指定扫描的时候按照什么规则排除那些组件，可以传入filter注解的数组
//includeFilters = Filter[] ：指定扫描的时候只需要包含哪些组件，可以传入filter注解的数组

//常用：FilterType.ANNOTATION：按照注解 如:@Controller
//常用：FilterType.ASSIGNABLE_TYPE：按照给定的类型，如：BookService.class，则该类和其子类都会被指定；
//FilterType.ASPECTJ：使用ASPECTJ表达式
//FilterType.REGEX：使用正则指定
//FilterType.CUSTOM：使用自定义规则
public class MainConfig {
   
   //给容器中注册一个Bean;类型为返回值的类型，id默认是用方法名作为id
   @Bean("person")
   public Person person01(){
      return new Person("lisi", 20);
   }

}
```

> 在JDK8之后ComponentScan是可重复注解，我们可以在一个类上重复添加，可以定义多个包扫描策略，或者使用ComponentScans注解，这个注解的值是ComponentScan[]

```java
@ComponentScans(
		value = {//数组类型，可以在里面定义多个ComponentScan注解
				@ComponentScan(value="com.atguigu",includeFilters = {
/*						@Filter(type=FilterType.ANNOTATION,classes={Controller.class}),
						@Filter(type=FilterType.ASSIGNABLE_TYPE,classes={BookService.class}),*/
						@Filter(type=FilterType.CUSTOM,classes={MyTypeFilter.class})
				},useDefaultFilters = false)	
		}
		)
```

![image-20211022214024859](https://image.imxyu.cn/file/image-20211022214024859.png)

**自定义TypeFilter指定包扫描规则**

```java
//所有类都会经过这个过滤器,进行判断false就是不加入到容器,true加入
public class MyTypeFilter implements TypeFilter {

   /**
    * metadataReader：读取到的当前正在扫描的类的信息，
    * metadataReaderFactory:可以获取到其他任何类信息的
    */
   @Override
   public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
         throws IOException {
      // TODO Auto-generated method stub
      //获取当前类注解的信息
      AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
      //获取当前正在扫描的类的类信息
      ClassMetadata classMetadata = metadataReader.getClassMetadata();
      //获取当前类资源（类的路径）
      Resource resource = metadataReader.getResource();
      
      String className = classMetadata.getClassName();
      System.out.println("--->"+className); //当前扫描类的名字
      if(className.contains("er")){ //如果包含了er，则加入到容器中
         return true;
      }
      return false; 
   }

}
```

#### 包扫描+注解注入对象

````java
@Controller
public class BookController {
}

````

```java
public interface BookService { //接口
}
```

```java
@Service
public class BookServiceImpl implements BookService {
}
```

```java
@Repository
public class BookDao {
}
```

### @Import

上面总结了两种方式

* 方式一：扫描自己写的类：包扫描+组件标注注解(@Controller/@Service/@Repository/@Component)

* 方式二：扫描导入的第三方包里边的组件：@Bean

* 方式三：快速的给容器中导入组件：@Import

#### @Import({组件1，组件2}) 导入组件

```java
@Configuration
@Import({Color.class, Animal.class}) //直接写要导入的组件
public class MyConfig2 {

    @Bean
    public Person person(){
        return new Person("zhangsan",34);
    }
    @Conditional({WindowsCondition.class})
    @Bean("bill")
    public Person person1(){
        return new Person("bill",34);
    }
    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person2(){
        return new Person("linus",34);
    }
}
```

#### ImportSelector

返回需要导入组件的全类名的数组

```java
public class MyImportSelector implements ImportSelector {
    /**
     * 自定义逻辑，返回需要导入的组件
     * @param importingClassMetadata 当前标注了@Import注解的类上的--所有注解信息--
     * @return 返回值就是要导入到容器中组件的全类名
     */
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        // 可返回空数组，但不能返回null
        return new String[]{"com.example.bean.Car","com.example.bean.Food"};
    }
}

```

在配置类上使用自定义的导入选择器

```java
@Configuration
=== 使用自定义的导入选择器 ===
@Import({Color.class, Animal.class, MyImportSelector.class})
public class MyConfig2 {

    @Bean
    public Person person(){
        return new Person("zhangsan",34);
    }
    @Conditional({WindowsCondition.class})
    @Bean("bill")
    public Person person1(){
        return new Person("bill",34);
    }
    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person2(){
        return new Person("linus",34);
    }
}
```

#### ImportBeanDefinitionRegistrar

自定义一个注册器

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    /**
     *
     * @param importingClassMetadata 当前标注了@Import注解的类上的--所有注解信息--
     * @param registry BeanDefinition注册类：把所有需要添加到容器中的bean，
     *                 调用BeanDefinitionRegistry.registerBeanDefinition()手动注册
     */
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        boolean color = registry.containsBeanDefinition("com.example.bean.Color"); //判断是否有这个bean
        boolean food = registry.containsBeanDefinition("com.example.bean.Food");
        // 判断容器中是否有指定的bean,有的话才会注册Book.class
        if (color&&food){
            // 注册bean时，可以指定bean的名称
            // 指定Bean定义信息：(Bean的类型、作用域等)
            RootBeanDefinition beanDefinition = new RootBeanDefinition(Book.class);
            registry.registerBeanDefinition("MyBook",beanDefinition);
        }
    }
}
```

在配置类上使用自定义的导入选择器MyImportBeanDefinitionRegistrar

```java
@Configuration
==== 使用自定义的MyImportBeanDefinitionRegistrar ===
@Import({Color.class, Animal.class, MyImportSelector.class, MyImportBeanDefinitionRegistrar.class})
===================================================
public class MyConfig2 {

    @Bean
    public Person person(){
        return new Person("zhangsan",34);
    }
    @Conditional({WindowsCondition.class})
    @Bean("bill")
    public Person person1(){
        return new Person("bill",34);
    }
    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person2(){
        return new Person("linus",34);
    }
}
```

### 使用FactoryBean

使用Spring提供的FactoryBean(工厂Bean)，区别于普通的bean。

源码：

```java
package org.springframework.beans.factory;
public interface FactoryBean<T> {
	T getObject() throws Exception;
	Class<?> getObjectType();
	boolean isSingleton();
}
```

自定义一个Color类的FactoryBean

```java
public class ColorFactoryBean implements FactoryBean<Color> {

    /**
     *
     * @return 返回一个Color对象，这个对象会返回到容器中
     * @throws Exception
     */
    public Color getObject() throws Exception {
        return new Color();
    }
    /**
     * 很重要
     * @return 将来通过该FactoryBean注入到容器中的bean，根据bean的id获取到的bean的类型就是该方法返回的类型
     */
    public Class<?> getObjectType() {
        return Color.class;
    }

    /**
     *
     * @return 如果返回值为true， 表示这是一个单实例对象，在容器中只会保存一份。
     *         如果返回值为false，表示这个是一个多实例对象，每次获取都会创建一个新的实例
     */
    public boolean isSingleton() {
        return true;
    }
}
```

```java
@Configuration
public class MyConfig2 {
    /**
     *
     * @return 将自己创建的Color类的FactoryBean注入到容器中
     */
    @Bean
    public ColorFactoryBean colorFactoryBean(){
        return new ColorFactoryBean();
    }
}
```

测试：

```java
 @Test
    public void test05(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        String[] beans = annotationConfigApplicationContext.getBeanDefinitionNames();
        for (String s : beans) {
            System.out.println(s);
        }
        // 工厂Bean获取的是调用getObject创建的对象
        Object bean1 = annotationConfigApplicationContext.getBean("colorFactoryBean");
        System.out.println("Color类的FactoryBean："+bean1.getClass());

        // 就要从容器中获取对应类的工厂bean
        Object bean2 = annotationConfigApplicationContext.getBean("&colorFactoryBean");
        System.out.println("Color类的FactoryBean："+bean2.getClass());
    }
```

输出

```java
myConfig2
colorFactoryBean

Color类的FactoryBean：class com.example.bean.Color

Color类的FactoryBean：class com.example.bean.ColorFactoryBean
```

> 这就是说，虽然在配置文件中装配的是ColorFactoryBean，但是按照ColorFactoryBean的id：colorFactoryBean从容器中获取到的bean的类型却是调用getObject创建的对象com.example.bean.Color。

**Spring与其它框架整合时，用的特别多，例如整合Mybaties。**

### 注册相关注解

####  @Scope，@Lazy

```java
@Configuration
public class MainConfig2 {
   
	/**
	 * @see ConfigurableBeanFactory#SCOPE_PROTOTYPE   任何环境都可以使用
	 * @see ConfigurableBeanFactory#SCOPE_SINGLETON   任何环境都可以使用
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST  request    只能在web容器里用
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION	 sesssion   只能在web容器里用
	 * 
	 * @Scope:调整作用域
	 * prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
	 * 					每次获取的时候才会调用方法创建对象；
	 * singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中。
	 * 			以后每次获取就是直接从容器（map.get()）中拿，
	 * request：同一次请求创建一个实例
	 * session：同一个session创建一个实例
	 *
	 * 默认是单实例的
	 * 
	 */
	@Scope("prototype") //多实例
	@Bean("person")
	public Person person(){
		  System.out.println("多实例模式下，开始向容器中添加Person组件...");
        return new Person("zhangsan",34);
	}

}
```

多实例测试：

```java
@Test
    public void test03(){
        // 读取通过注解注入容器中的实例对象
        ApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        Person person1 = (Person) annotationConfigApplicationContext.getBean("person");
        Person person2 = (Person) annotationConfigApplicationContext.getBean("person");
        System.out.println(person1==person2);
        System.out.println("===========");
        === 输出结果为false，也就是说两次获取的对象组件是不一样的。
    }
```

单实例：

```java
@Configuration
public class MainConfig2 {
   
   /**
    * 
    * 懒加载：
    *        单实例bean：默认在容器启动的时候创建对象；
    *        单实例+懒加载：容器启动不创建对象。第一次使用(获取)Bean创建对象，并初始化；
    * 
    */
   @Lazy
   @Scope("singleton") //不定义的话默认就是这个单实例
   @Bean("person")
   public Person person(){
      System.out.println("给容器中添加Person....");
      return new Person("张三", 25);
   }
```

#### @Conditional

按照一定的条件进行判断，满足条件时才给容器中注册bean

源码：

![@conditional](https://image.imxyu.cn/file/@conditional.png)

【案例】根据不同的操作系统注入组件

补充知识点：根据条件上下文对象获取上下文环境

```java
@Configuration
public class MyConfig2 {

    @Bean
    public Person person(){
        return new Person("zhangsan",34);
    }
    
    //在配置类中使用@Conditional条件进行判断
    //=== 条件中传入自定义的接口作判断，是Windows系统才注入 ===
    @Conditional({WindowsCondition.class})  
    @Bean("bill")
    public Person person1(){
        return new Person("bill",34);
    }
    
    //===  条件中传入自定义的接口作判断，是Linux系统才注入 ===
    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person2(){
        return new Person("linus",34);
    }
}
```

自定义接口：LinuxCondition

```java
public class LinuxCondition implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 获取当前环境信息
        Environment environment = context.getEnvironment();
        String osName = environment.getProperty("os.name");
        return osName.contains("Linux");
    }
}
```

自定义接口：WindowsCondition

```java
public class WindowsCondition implements Condition {
    /**
     *
     * @param context 判断条件能使用的上下文(环境)
     * @param metadata 注释信息
     * @return boolean
     */
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 获取到ioc使用的beanFactory(创建对象以及进行装配的工厂)
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        // 获取类加载器
        ClassLoader classLoader = context.getClassLoader();
        // 获取到bean定义的注册类，所有的bean的定义都在这个里面进行注册
        BeanDefinitionRegistry registry = context.getRegistry();
        // 获取当前环境信息
        Environment environment = context.getEnvironment();
        String osName = environment.getProperty("os.name");
        return osName.contains("Windows");
    }
}
```

## Bean的生命周期

Bean的生命周期由容器管理，即管理bean的：创建 -- 初始化 -- 销毁。我们可以自定义初始化和销毁方法，容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法。

### @Bean指定初始化和销毁方法

1. 在实体类中创建初始化和销毁方法

```java
public class Car {
    public Car(){
        System.out.println("car contructor ...");
    }

    public void init(){
        System.out.println("Car初始化");
    }
    public void destory(){
        System.out.println("Car销毁");
    }
}
```

2. 在配置文件中，注入bean时指定初始化和销毁方法

```java
@Configuration
public class MyConfig2 {
    @Bean(initMethod = "init",destroyMethod = "destory")
    public Car car(){
        return new Car();
    }
}
```

测试：

```java
    @Test
    public void test06(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        // 关闭容器的时候调用bean的销毁方法
        annotationConfigApplicationContext.close();
    }

输出：
car contructor ...
Car初始化
Car销毁
```

**要注意的是，单实例和多实例的初始化和销毁时机。**

**初始化**

**对象创建完成 ，并赋值好（构造函数），调用初始化方法**

**销毁：**

**单实例：容器关闭的时候调用销毁方法。**

**多实例：容器不会管理整个bean，也就是说容器不会调用销毁方法，需要手动调用。**

### 实体类实现接口

通过让bean实现**InitialzingBean**接口来定义初始化逻辑，实现**DisposableBean**接口来定义销毁逻辑。

1. 自定义一个Build类，并实现InitializingBean和DisposableBean接口

```java
public class Build implements InitializingBean, DisposableBean {

    public Build(){
        System.out.println("Build 构造函数...");
    }

    /**
     * 单实例Bean的销毁方法，在容器关闭的时候进行调用
     * @throws Exception 销毁失败抛异常
     */
    public void destroy() throws Exception {
        System.out.println("Build的销毁方法...");
    }

    /**
     * Bean初始化方法
     * @throws Exception 初始化失败抛异常
     */
    public void afterPropertiesSet() throws Exception {
        System.out.println("Build的初始化方法");
    }
}
```

2. 在配置类中注入Build的bean

```java
@Configuration
public class MyConfig2 {
    @Bean
    public Build build(){
        return new Build();
    }
}
```

省略测试....

### 实体类注解指定(使用JSR250规范)

@PostConstruct：在bean创建完成并且属性值赋值完成后执行初始化；

@PreDestroy：在容器销毁bean之前，通知容器进行清理工作；

1. 创建一个Dog类，并在初始化和销毁方法上添加对应的注解

```java
public class Dog {
    public Dog(){
        System.out.println("Dog 的构造函数");
    }

    /**
     * 对象创建并赋值后，调用
     */
    @PostConstruct
    public void init(){
        System.out.println("Dog构造之后调用...@PostConstruct");
    }

    /**
     * 对象移除之前，调用
     */
    @PreDestroy
    public void destory(){
        System.out.println("Dog对应的bean移除之前调用...@PreDestroy");
    }
}
```

2. 在配置类中注入Dog对应的bean

```java
@Configuration
public class MyConfig2 {
    @Bean
    public Dog dog(){
        return new Dog();
    }
}
```

测试：

```java
    @Test
    public void test06(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfig2.class);
        // 关闭容器的时候调用bean的销毁方法
        annotationConfigApplicationContext.close();
    }
输出结果：
Dog 的构造函数
Dog构造之后调用...@PostConstruct
Dog对应的bean移除之前调用...@PreDestroy
```

### BeanPostProcessor后置处理器

在bean的**初始化方法之前**和**初始化方法之后**进行一些处理工作。注意是初始化方法，和销毁方法没有一毛钱的关系。

**初始化方法之前:   调用：**postProcessBeforeInitialization()

**初始化方法之后：调用：**postProcessAfterInitialization()

**即使没有自定义初始化方法，在组件创建前后，后置处理器方法也会执行**

1. 自定义类实现BeanPostProcessor 接口

```java
public class MyBeanPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization"+"--"+beanName+"--"+bean);
        return bean;
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization"+"--"+beanName+"--"+bean);
        return bean;
    }
}
```

2. 在配置文件中注入该后置处理器bean类

```java
@Configuration
public class MyConfig2 {
	@Bean
    public MyBeanPostProcessor myBeanPostProcessor(){
        return new MyBeanPostProcessor();
    }
}
```

输出

```java
car的构造函数 ...
postProcessBeforeInitialization--car--com.example.bean.Car@7c417213
Car的初始化方法
postProcessAfterInitialization--car--com.example.bean.Car@7c417213
Car的销毁方法
```

## 属性赋值相关的注解

#### @Value

```java
 /**
     * 使用@Value赋值
     *    1、基本数值
     *    2、可以些SpEL，#{}
     *    3、可以写${},取出配置文件中的值(即在运行环境变量中的值).
     *      通过@PropertySource注解将properties配置文件中的值存储到Spring的 Environment中，
     *      Environment接口提供方法去读取配置文件中的值，参数是properties文件中定义的key值。
     */
```

通过@PropertySource注解将properties配置文件中的值存储到Spring的 Environment中，Environment接口提供方法去读取配置文件中的值，参数是properties文件中定义的key值。

Person类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    @Value("张三")
    private String name;
    @Value("#{20-2}")
    private Integer age;
    @Value("${person.nickName}")
    private String nickName;
}
```

person.properties

```yml
person
	nickName="zhangzhang"
```

使用PropertySources读取外部配置文件中的属性k/v保存到运行的环境变量中

```java
@PropertySources(@PropertySource(value = {"classpath:/person.properties"}))
@Configuration
public class MyConfigOfPropertyValues {
    @Bean
    public Person person(){
        return new Person();
    }
}
```

测试

```java
    @Test
    public void test07(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MyConfigOfPropertyValues.class);
        Person person2 = (Person) annotationConfigApplicationContext.getBean("person");
        System.out.println(person2);

    }
//输出：Person(name=张三, age=18, nickName="zhangzhang")
```

从环境变量中获取也是可以的

## 自动装配

#### @Autowired

1. 默认优先按照类型去容器中找对应的组件：applicationContext.getBean(BookServiceImpl.class);
2. 如果找到多个相同类型的组件，将会按照属性的名称作为组件的id去容器中查找。

![](https://image.imxyu.cn/file/autowired.png)htt



##### @Autowired标注在方法头上

![autowired_method1](https://image.imxyu.cn/file/autowired_method1.png)

测试

![autowired_method2](https://image.imxyu.cn/file/autowired_method2.png)

##### @Autowired标注在构造器上

如果组件只有一个参数构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取。

![autowired_construct](https://image.imxyu.cn/file/autowired_construct.png)

##### @Autowired标注在参数位置

![autowired_param1](https://image.imxyu.cn/file/autowired_param1.png)

![autowired_param1](https://image.imxyu.cn/file/autowired_param2.png)

#### @Autowired和@Qualifier配合

![Qualifier](https://image.imxyu.cn/file/Qualifier.png)

自动装配的前提，默认一定要将容器的组件赋好值。否则没有就会报错。

是否可以根据容器中组件的有无来判断是否进行装配呢？

![autowired2](https://image.imxyu.cn/file/autowired2.png)

#### @Primary设置首选bean

让spring进行自动装配的时候，默认使用首选的bean。

![primary](https://image.imxyu.cn/file/primary.png)

#### @Resource和@Inject

1. @Resource属于JSR250规范，@Inject属于JSR330规范。都是java规范。

@Resource和@Autowired一样实现自动装配功能，默认是按照1.类型 2.组件名称进行装配，也可以指定要装配组件的名称。但其不支持@Primary和@Autowired(required=false)的功能。

2. @Inject需要导入注解

```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

支持@Autowired(required=false)

**推荐使用spring自带的**。

## @Profile

 Profile: 指定组件在哪个环境中的情况下才能被注册到容器中，不指定任何环境下都能指定。
 加了环境标识的bean，只有对应的环境被激活了，才会注册到容器中。但是如果标了default，则会默认加载这个。
*
 Spring为我们提供的可以根据当前环境，动态的激活和切换一系列bean组件的功能
 我们有开发环境、测试环境、生成环境
 对应的数据源有：(/A)(/B)(/C)
 利用注解：@Profile



@Profile也可以写在类上，表明只有在指定的环境下，类中的内容才会被激活使用。

### 【案例】根据环境切换数据源

1. 先加入数据库连接池依赖

```xml
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.22</version>
        </dependency>
```

2. 配置一个数据库链接的配置文件

```xml
db.user=root
db.password=root
db.driverClass=com.mysql.jdbc.Driver
```

3. 在配置类中配置不同环境使用的数据源

```java
@PropertySource("classpath:/dbconfig.properties") //将配置文件以键值对的形式加入到spring环境变量中，可以使用${}表达式获取
@Configuration
public class MainConfigOfProfile implements EmbeddedValueResolverAware {

    @Value("${db.user}")
    private String user;

    private StringValueResolver valueResolver;

    private String driverClass;

    @Profile("test")
    @Bean("testDatasource")
    public DataSource dataSourceTest(@Value("${db.password}") String password) throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(password);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");

        // 利用值解析器
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }
    @Profile("dev")
    @Bean("devDatasource")
    public DataSource dataSourceDev() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword("root");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/dev");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }
    @Profile("prod")
    @Bean("prodDatasource")
    public DataSource dataSourceProd() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("root");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }
    @Profile("default") //默认使用这个
    @Bean("prodDatasource")
    public DataSource dataSourceDefault() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("root");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }

    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        // 利用值解析器，解析字符串
        this.valueResolver = resolver;
        this.driverClass = valueResolver.resolveStringValue("${db.driverClass}");
    }
}
```

如何切换环境呢？

/**

   \* 切换环境的方式：

   \*  1、使用命令行动态参数：在虚拟机参数位置加载-Dspring.profiles.active=test(test是测试的环境标识)

   \*  2、代码的方式激活某种环境

   */

```java
@Test
public void test01(){
    // 1、创建一个applicationContext
    AnnotationConfigApplicationContext applicationContext =
        new AnnotationConfigApplicationContext();
    // 2、设置需要激活的环境,可以设置多个
    applicationContext.getEnvironment().setActiveProfiles("test","dev");
    // 3、注册配置类
    applicationContext.register(MainConfigOfProfile.class);
    // 4、启动刷新容器
    applicationContext.refresh();

    String[] beanNamesForType = applicationContext.getBeanNamesForType(DataSource.class);
    for (String s : beanNamesForType) {
        System.out.println(s);
    }
    applicationContext.close();
}
```

> 使用之前说的@Conditional 也可以实现，当然@Conditional注解，可以自定义任意将组件加入到容器的规则

