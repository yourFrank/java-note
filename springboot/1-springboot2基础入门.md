视频地址：https://www.bilibili.com/video/BV19K4y1L7MT?from=search&seid=5309862794027495419&spm_id_from=333.337.0.0

雷丰阳老师语雀笔记地址：https://www.yuque.com/atguigu/springboot/na3pfd

# springboot2基础入门

springboot 学习参考官方文档即可

![image.png](https://image.imxyu.cn/file/1602654700738-b6c50c90-0649-4d62-98d3-57658caf0fdb.png)

![image.png](https://image.imxyu.cn/file/1602654837853-48916a4f-cb5a-422c-ba7a-83b027c5bf24.png)

## 查看新版本特性

在官网中点进github地址

![image-20220312105208779](https://image.imxyu.cn/file/image-20220312105208779.png)

在readme中有更新的特性简介

从1-2 是一个大版本，我们可以点击进去看看改了什么，2.1-2.2 都是小版本的更新

![image-20220312105232271](https://image.imxyu.cn/file/image-20220312105232271.png)

## getting start

参考spring官方文档即可

https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started

对应雷老师笔记

https://www.yuque.com/atguigu/springboot/lcfeme

# 自动配置原理

# 1、SpringBoot特点



## 1.1、依赖管理

- 父项目做依赖管理

首先点击这个parent 进去看一下

![image-20220312105744083](https://image.imxyu.cn/file/image-20220312105744083.png)

点击进去还有一个父项目，这个里面就是保存了所有的依赖版本号的管理，自动版本仲裁机制

![image-20220312105725010](https://image.imxyu.cn/file/image-20220312105725010.png)

在这个properties 标签中包含了所有的版本号声明

![image-20220312110042867](https://image.imxyu.cn/file/image-20220312110042867.png)

- 开发导入starter场景启动器

```xml
1、见到很多 spring-boot-starter-* ： *就某种场景 （官方的）
2、只要引入starter，这个场景的所有常规需要的依赖我们都自动引入
3、SpringBoot所有支持的场景
https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter
4、见到的  *-spring-boot-starter： 是第三方为我们提供的简化开发的场景启动器。
5、所有场景启动器最底层的依赖，所有的starter启动器底层都有一个这个依赖
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <version>2.3.4.RELEASE</version>
  <scope>compile</scope>
</dependency>
```

- 无需关注版本号，自动版本仲裁（自动使用父类中properties 标签中声明的版本号）

```xml
1、引入依赖默认都可以不写版本
2、引入非版本仲裁的jar，要写版本号（父类里面没有声明的）。
```

- 可以修改默认版本号

```xml
1、首先查看spring-boot-dependencies里面规定当前依赖的版本 
2、在pom 文件中自己写一个properties标签，指明想要的依赖的版本号。
    <properties>
        <mysql.version>5.1.43</mysql.version>
    </properties>
```

> 或者直接在引入的时候写好版本

举例： 更改mysql 版本为5.7

方式1、首先在父类中查看一下声明的版本号

![image-20220312110239429](https://image.imxyu.cn/file/image-20220312110239429.png)

方式2、复制这行，到自己的pom 文件中自定义properties 标签重写这个版本号。然后刷新maven重新导入

![image-20220312110415082](https://image.imxyu.cn/file/image-20220312110415082.png)

此时我们引入的mysql 版本就变成我们自定义的版本号了

![image-20220312110645767](https://image.imxyu.cn/file/image-20220312110645767.png)

或者我们可以直接在引入的时候直接写好version的版本，这样也是可以的

![image-20220312115823633](https://image.imxyu.cn/file/image-20220312115823633.png)

> 这里利用的都是maven的就近原则，要是在自己的这个xml中定义了properties 就会用自己的，否则就会依赖传递用父类的版本

## 1.2、starter自动配置

点开我们引入的starter-web为例，我们看一下里面

![image-20220312120056976](https://image.imxyu.cn/file/image-20220312120056976.png)

![image-20220312120134711](https://image.imxyu.cn/file/image-20220312120134711.png)

### 自动配好Tomcat

- - 引入Tomcat依赖。
  - 配置Tomcat

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <version>2.3.4.RELEASE</version>
      <scope>compile</scope>
    </dependency>
```

### 自动配好SpringMVC

- - 引入SpringMVC全套组件
  - 自动配好SpringMVC常用组件（功能）

### 自动配好Web常见功能

​			如：字符编码问题

- - SpringBoot帮我们配置好了所有web开发的常见场景

### 默认的包结构或@ComponentScan指定扫描

- - **主程序所在包及其下面的所有子包里面的组件都会被默认扫描进来**
  - 无需以前的包扫描配置
  - 想要改变扫描路径，@SpringBootApplication(scanBasePackages=**"com.atguigu"**)

- - - 或者@ComponentScan 指定扫描路径

```java
@SpringBootApplication
等同于
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.atguigu.boot")
```

> 注意1、这里如果没有定义包扫描路径，放在了主程序类之外的包是不会被扫描的
>
> 2、主程序类不能放在java默认包下否则会报错，需要放在创建的包下

![image-20220312125610436](https://image.imxyu.cn/file/image-20220312125610436.png)

### 各种配置拥有默认值

- - 默认配置最终都是映射到某个类上，如：MultipartProperties
  - 配置文件的值最终会绑定每个类上，这个类会在容器中创建对象



### 按需加载所有自动配置项

我们之前说过其他的都是以spring-boot-starter这个为基础，也就是说这个是必须有的，我们看一下这个spring-boot-starter里面
![image-20220312120248103](https://image.imxyu.cn/file/image-20220312120248103.png)

- - 非常多的starter
  - 引入了哪些场景这个场景的自动配置才会开启
  - SpringBoot所有的自动配置功能都在 spring-boot-autoconfigure 包里面

这个自动配置包里里面包含了所有场景的自动配置，但是这些场景不会全部生效，

- ![image-20220312120446801](https://image.imxyu.cn/file/image-20220312120446801.png)

我们随便打开一个看看，这些类里面都会判断，只有我们引入了相关的依赖，才会生效，所以说不是全生效的

![image-20220312120707979](https://image.imxyu.cn/file/image-20220312120707979.png)

- ......

# 2、容器功能

## 2.1、组件添加

这个注解就相当于一个xml 文件，说明这是一个配置类

### 1、@Configuration

- 基本使用
- Springboot2新特性：配置类可以指定**Full全模式与Lite轻量级模式**

- - 示例
  - 最佳实战

- - - 配置 类组件之间无依赖关系用Lite模式(proxyBeanMethods = false)加速容器启动过程，减少判断
    - 配置类组件之间有依赖关系 Full(proxyBeanMethods = true)，方法会被调用得到之前单实例组件，用Full模式

```java

/**
 * 1、配置类里面使用@Bean标注在方法上给容器注册组件，默认也是单实例的
 * 2、配置类本身也是组件
 * 3、proxyBeanMethods：代理bean的方法
 *      Full(proxyBeanMethods = true)（保证每个@Bean方法被调用多少次返回的组件都是单实例的（默认）
 *      Lite(proxyBeanMethods = false)（每个@Bean方法被调用多少次返回的组件都是新创建的）
 */
@Configuration(proxyBeanMethods = false) //告诉SpringBoot这是一个配置类 == 配置文件
public class MyConfig {

    /**
     * Full:外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
     * @return
     */
    @Bean //给容器中添加组件。以方法名作为组件的id。返回类型就是组件类型。返回的值，就是组件在容器中的实例
    public User user01(){
        User zhangsan = new User("zhangsan", 18);
        //user组件依赖了Pet组件
        zhangsan.setPet(tomcatPet());
        return zhangsan;
    }

    @Bean("tom") //@Bean 中可以指定bean的名字
    public Pet tomcatPet(){
        return new Pet("tomcat");
    }
}


```

@Configuration测试代码如下:

通过 SpringApplication.run(MainApplication.class, args); 的返回值就是我们的Ioc 容器

#### proxyBeanMethods = true

```java
@Configuration(proxyBeanMethods = true)  //默认就为true，可以不写
public class MyConfig {
```

```java
@SpringBootApplication
public class SpringbootApplication {

    public static void main(String[] args) {
        //1、返回我们IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(SpringbootApplication.class, args);

        //2、查看容器里面的组件
//        String[] names = run.getBeanDefinitionNames();
//        for (String name : names) {
//            System.out.println(name);
//        }

        //查看配置类的类型，此时为com.ty.config.MyConfig$$EnhancerBySpringCGLIB$$4ea35f43@4a183d02
        MyConfig config = run.getBean(MyConfig.class);
        System.out.println(config);

        //3、从容器中获取宠物，看这两个宠物是否是同一个宠物
        Pet tom01 = run.getBean("tom",Pet.class);
        Pet tom02 = run.getBean("tom", Pet.class);
        System.out.println(tom01==tom02); //true

        //此时拿到一个用户
        User user01 = run.getBean(User.class);
        //判断这个用户注入的宠物看是否是容器中的宠物
        System.out.println(user01.getPet()==tom01);  //true
    }
}
```

> 当proxyBeanMethods = true 时，配置类的类型为spring通过cglib生成的代理类型对象com.ty.config.MyConfig$$EnhancerBySpringCGLIB$$4ea35f43@4a183d02
>
> 此时在这个配置类中调用@Bean 方法的组件都会先检查看是否有，如果有直接返回（保证每个调用@Bean方法返回的组件都是单实例的）
>
> 此时因为配置的为true，所以使用的spring的代理配置类，这里使用 zhangsan.setPet(tomcatPet()); 因为tomcatPet()是标注了@Bean方法的组件，所以会从容器中找到这个组件注入进去，保证每个用户的这个pet 都是单例的



#### proxyBeanMethods = false

```java
@Configuration(proxyBeanMethods = false)  //当我们设置成false
public class MyConfig {
```

```java
@SpringBootApplication
public class SpringbootApplication {

    public static void main(String[] args) {
        //1、返回我们IOC容器
        ConfigurableApplicationContext run = SpringApplication.run(SpringbootApplication.class, args);

        //2、查看容器里面的组件
//        String[] names = run.getBeanDefinitionNames();
//        for (String name : names) {
//            System.out.println(name);
//        }

        //查看配置类的类型，此时为com.ty.config.MyConfig@6dc1484
        MyConfig config = run.getBean(MyConfig.class);
        System.out.println(config);

        //3、从容器中获取宠物，看这两个宠物是否是同一个宠物
        Pet tom01 = run.getBean("tom",Pet.class);
        Pet tom02 = run.getBean("tom", Pet.class);
        System.out.println(tom01==tom02); //true

        //此时拿到一个用户
        User user01 = run.getBean(User.class);
        //判断这个用户注入的宠物看是否是容器中的宠物
        System.out.println(user01.getPet()==tom01);  //false
    }
}
```

> 此时当我们指定为false之后，这个配置类对象的类型变成了我们的类型，也就是说spring并没有为我们创建代理对象。此时调用zhangsan.setPet(tomcatPet());  通过tomcatPet() 这个方法会直接返回一个新的宠物对象，也就是说不会从容器中检查（速度快）。 因此每个用户里的都是新的宠物对象

- 总结
  - 配置 类组件之间**无依赖关系**用Lite模式加速容器启动过程，减少判断
  - 配置 类组件之间**有依赖关系**，方法会被调用得到之前单实例组件，用Full模式（默认）（依赖的对象注入的都是容器中同一个对象）



> 注意这里说的单例指的是一个对象通过依赖注入的另一个标注有@Bean方法的对象（比如说用户注入宠物，此时的用户中的宠物对象就是同一个/单例的），而对于单独宠物这个对象本身在容器中还是单例的存在。只是在用户注入宠物的这个过程中使用的宠物没有检查容器直接new 返回了一个新的，所以速度比较快

### 2、给容器中导入组件

@Bean、@Component、@Controller、@Service、@Repository，它们是Spring的基本标签，在Spring Boot中并未改变它们原来的功能。

> 使用@Component、@Controller、@Service、@Repository 注解注册的id 名是它的类名, 此时 id 名对应的就是Person

```java
@Component
@Data
public class Person {
}
```

给容器中注册组件的方法和之前一样，这几个注解都可以，只要在包扫描的范围内就行

使用@Import 注解标注在类上也可以指定注入某个类型的组件，同样的@Import 可以标注在任何一个类上（只要在包扫描范围就行，不一定非要标注在主配置类中）



代码： 注入一个User类型的组件，和一个我们引入的logback的第三方jar包的DelayingShutdownHook类

```java
 * 4、@Import({User.class, DelayingShutdownHook.class})
 *      给容器中自动创建出这两个类型的组件、默认组件的名字就是全类名（包名+类名）
 */

@Import({User.class, DelayingShutdownHook.class})
@Configuration(proxyBeanMethods = false) //告诉SpringBoot这是一个配置类 == 配置文件
public class MyConfig {
}
```

主配置类中运行

```java
String[] beanNamesForType2 = run.getBeanNamesForType(User.class);  //获取所有类型的bean，因为此时user类型的不止一个。上面通过@bean注册过
        for (String s : beanNamesForType2) {
            System.out.println(s);
        }
        DelayingShutdownHook bean = run.getBean(DelayingShutdownHook.class);
        System.out.println(bean);
```

两个user组件，其中通过@Bean注册的id名字是它的属性名，通过import 注册的组件的id是它的全类名

![image-20220312142203994](https://image.imxyu.cn/file/image-20220312142203994.png)

![image-20220312142347019](https://image.imxyu.cn/file/image-20220312142347019.png)



总结：

* @Component、@Controller、@Service、@Repository 注册的bean 的id 名为类名，任意类（除非是第三方jar包的类，不能我们修改，此时只能用下面两种方式）
* @Import注册的bean 名为全类名（包+类），使用范围可以在任意类上
* @Bean 注册的bean组件名为方法名， 使用范围在标有@Configuration 的注解类上



### 3、条件装配 @Conditional

**条件装配：满足Conditional指定的条件，则进行组件注入**

可以看到这个组件时可以放到方法上，也可以放到类上

![image-20220312143750593](https://image.imxyu.cn/file/image-20220312143750593.png)

![在这里插入图片描述](https://image.imxyu.cn/file/20210205005453173.png)

* 这里我们以@ConditionalOnBean 来举例
* 当我们放到方法上时，表示如果我们满足了@ConditionalOnBean () 中的条件才进行装配

下面就表示容器中如果有tom这个组件的话才进行装配User

![image-20220312144201512](https://image.imxyu.cn/file/image-20220312144201512.png)



* 当我们放到类上时

表明满足这个条件的话，下面所有的组件才进行装配

![image-20220312144327409](https://image.imxyu.cn/file/image-20220312144327409.png)

> 其他的@Conditional 类型的注解也都一样，见名知意，这里就不一 一举例了

## 2.2、原生xml配置文件引入

### @ImportResource导入配置文件

比如说此时我们以前有xml 配置文件的方式导入我们的组件，或者很多老的框架只支持使用xml 配置文件的方式导入组件，此时我们可以使用这个注解@ImportResource将我们的xml 文件进行导入解析创建组件

![image-20220312144937696](https://image.imxyu.cn/file/image-20220312144937696.png)

比如说上面这个xml 文件，此时spring是不会进行解析的，它不知道这个xml 文件是干什么的。此时这两个对象是不会被加入到容器中的

此时我们在配置类上使用这个注解

```java
@ImportResource("classpath:beans.xml")
@Configuration()
public class MyConfig {}

======================测试=================
        boolean haha = run.containsBean("haha");
        boolean hehe = run.containsBean("hehe");
        System.out.println("haha："+haha);//true
        System.out.println("hehe："+hehe);//true
```

此时容器中就有了这两个组件

## 2.3、配置属性绑定值

之前我们读取配置文件绑定到一个对象上可能需要下面这样的代码（比如之前jdbc获取数据库连接信息的时候，是很常见的）

```java
public class getProperties {
     public static void main(String[] args) throws FileNotFoundException, IOException {
         Properties pps = new Properties();
         pps.load(new FileInputStream("a.properties"));
         Enumeration enum1 = pps.propertyNames();//得到配置文件的名字
         while(enum1.hasMoreElements()) {
             String strKey = (String) enum1.nextElement();
             String strValue = pps.getProperty(strKey);
             System.out.println(strKey + "=" + strValue);
             //封装到JavaBean。
         }
     }
 }
```

此时我们可以通过springboot 中的注解来实现这个功能



首先我们需要在spring的配置文件上写上我们需要绑定的属性key 和value

![image-20220312150050935](https://image.imxyu.cn/file/image-20220312150050935.png)

### 1、@Component+@ConfigurationProperties

这两个注解都是在组件上标注的

```java
/**
 * 只有在容器中的组件，才会拥有SpringBoot提供的强大功能
 */
@Component
//指定绑定的前缀名，此时spring就会按照这个类中的前缀+属性和我们的配置文件中的key相对应上
@ConfigurationProperties(prefix = "mycar")
public class Car {

    private String brand;
    private Integer price;

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public Integer getPrice() {
        return price;
    }

    public void setPrice(Integer price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "Car{" +
                "brand='" + brand + '\'' +
                ", price=" + price +
                '}';
    }
}
```

### 2、@EnableConfigurationProperties + @ConfigurationProperties

在配置类上标注@EnableConfigurationProperties ，在组件上标注@ConfigurationProperties 来指定前缀名

配置类：

```java
@EnableConfigurationProperties(Car.class)
//这个注解有两个功能：
//1、开启Car配置绑定功能
//2、把这个Car这个组件自动注册到容器中
public class MyConfig {
}

```

这两种方式适用于不同的情况：

第一种方式是我们可以修改类的情况下

第二种方式是如果我们要给第三方jar包中的属性赋值的话，就可以用在配置类上标注解的方式



### 3、@PropertyResource 、@Value

@PropertyResource: 读取properties属性配置文件。 使用属性配置文件可以实现外部化配置 ，

在程序代码之外提供数据。



步骤：

1. 在resources目录下，创建properties文件， 使用k=v的格式提供数据
2. 在PropertyResource 指定properties文件的位置
3. **使用@Value（value="${key}"）**



```java
@Configuration
@ImportResource(value ={ "classpath:applicationContext.xml","classpath:beans.xml"})
@PropertySource(value = "classpath:config.properties")
@ComponentScan(basePackages = "com.bjpowernode.vo")
public class SpringConfig {
}
```

> 如果我们的定义的属性是在主配置文件，也就是application.properties /application.yml 中的，可以直接使用@Value手动赋值，不需要@PropertiesResouce引入配置文件



# 3、自动配置原理

## 3.1、引导加载自动配置类

我们之前说过@SpringBootApplication 注解是由这三个注解合成的，那么我们一个个来看一下这三个注解

![image-20220312151028983](https://image.imxyu.cn/file/image-20220312151028983.png)

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication{}

======================
    
```

### 1、@SpringBootConfiguration

@Configuration。代表当前是一个配置类

### 2、@ComponentScan

指定扫描哪些，Spring注解；

@ComponentScan 扫描器，找到注解，根据注解的功能创建对象，给属性赋值等等。
默认扫描的包： @ComponentScan所在的类所在的包和子包。

### 3、@EnableAutoConfiguration

我们分别看一下这两个注解干了什么？

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

#### 1、@AutoConfigurationPackage

首先第一个注解@AutoConfigurationPackage

自动配置包？指定了默认的包规则

```java
@Import(AutoConfigurationPackages.Registrar.class)  //给容器中导入一个组件
public @interface AutoConfigurationPackage {}

//利用Registrar给容器中导入一系列组件
//将指定的一个包下的所有组件导入进来？MainApplication 所在包下。
```

可以看到使用@Import 注入的这个组件实现了这个方法，

这个方法可以获取我们的元注解信息，也就是我们最外层的注解@SpringBootApplication  这个注解的信息，通过这个注解获取到这个注解所在的类的包名，然后将我们的这个包下的所有类组件注册进来

![image-20220312151316484](https://image.imxyu.cn/file/image-20220312151316484.png)

![image-20220312151656850](https://image.imxyu.cn/file/image-20220312151656850.png)



这就是我们说的为什么我们可以默认在主配置类所在的包或者子包下创建组件标注注解，不需要写@ComponentScan 就可以帮我们注册进去的原因



#### 2、@Import(AutoConfigurationImportSelector.class)

我们点进去这个AutoConfigurationImportSelector.class ，里面有这个selectImport方法，调用了下面的这个getAutoConfigurationEntry方法，就是获取自动配置类的属性



![image-20220312153032570](https://image.imxyu.cn/file/image-20220312153032570.png)

我们看一下这个方法，里面有一个获取候选类的方法

![image-20220312153137926](https://image.imxyu.cn/file/image-20220312153137926.png)

继续调用

![image-20220312153231761](https://image.imxyu.cn/file/image-20220312153231761.png)

![image-20220312153304050](https://image.imxyu.cn/file/image-20220312153304050.png)



在这个方法中看到了，加载了这个资源配置文件

![image-20220312153323050](https://image.imxyu.cn/file/image-20220312153323050.png)



就会从我们的类路径下寻找这个配置文件

![image-20220312153346570](https://image.imxyu.cn/file/image-20220312153346570.png)

我们的自动配置这个jar包中包含了这个配置文件，里面写了所有自动配置类的信息

![image-20220312153437036](https://image.imxyu.cn/file/image-20220312153437036.png)



```java
1、利用getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件
2、调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取到所有需要导入到容器中的配置类
3、利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)；得到所有的组件
4、从META-INF/spring.factories位置来加载一个文件。
	默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件
    spring-boot-autoconfigure-2.3.4.RELEASE.jar包里面也有META-INF/spring.factories
    
```

一共加载了127个自动配置组件的信息，但是这127个不会全部生效

![image.png](https://image.imxyu.cn/file/1602845382065-5c41abf5-ee10-4c93-89e4-2a9b831c3ceb.png)

## 3.2、按需开启自动配置项

虽然我们127个场景的所有自动配置启动的时候默认全部加载。xxxxAutoConfiguration
按照条件装配规则（@Conditional），最终会按需配置。

这个方法后面得到127个之后还要进行过滤排除

![image-20220312153137926](https://image.imxyu.cn/file/image-20220312153137926.png)

比如说这种的，需要我们引入一些jar包对应的类才能开启的，都会在这个条件注解上标明开启的条件。

![image-20220312154041032](https://image.imxyu.cn/file/image-20220312154041032.png)

只有满足条件的自动配置类才会加入到容器中，然后按照类中的@Bean为我们注入一些组件

比如说我们的DispatcherServletAutoConfiguration 这个自动配置类，就满足了条件，就会今夏下面的注册组件等一系列操作，

我们看到会为我们new 一个dispatcherServet（）组件放到容器中

并且会为我们进行参数绑定，这个注解我们之前说过

```
@EnableConfigurationProperties(WebMvcProperties.class)
```

![image-20220312154443771](https://image.imxyu.cn/file/image-20220312154443771.png)

@EnableConfigurationProperties(WebMvcProperties.class)

@ConfigurationProperties(prefix = "spring.mvc") ，这两个注解可以将我们配置文件中的值绑定到这个类上，**同时还会把我们这个WebMVcProperties 组件注册到我们的容器中**

![image-20220312154517051](https://image.imxyu.cn/file/image-20220312154517051.png)

并且如果我们的@bean方法中是一个对象的话，会在容器中帮我们找到这个组件，然后装配到我们的参数上去

![image-20220312154623315](https://image.imxyu.cn/file/image-20220312154623315.png)



## 3.3、修改默认配置名

在dispatcherservletConfigure 自动配置类中还有一个比较有意思的，就是帮我们改名字（也不是改就是添加一个规范的名字）

@ConditionalOnBean(MultipartResolver.class) 首先会判断我们容器中是否有这个文件解析的组件，

@ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME)，接下来会判断我们的这个文件上传的组件名是否叫这个规定的multipartResolver

如果我们容器中有这个文件上传组件，并且我们的名字不叫这个的话，就会进入这个方法，会把我们的组件return 

> 我们上面说过通过参数可以拿到我们容器中这个类型的组件，然后在@Bean中通过return 对象的话，会加入到容器中，而这个组件的名字就是以方法名为命名的，此时我们容器中就会有这个multipartResolver 类型的组件了。这就是spring 防止我们瞎起名然后找不到默认的组件 

![image-20220312154705472](https://image.imxyu.cn/file/image-20220312154705472.png)

![image-20220312154817567](https://image.imxyu.cn/file/image-20220312154817567.png)



比如说我们的乱码的问题（如果没配置的话如果我们在url 请求路径上写中文后台得到是会乱码的），spring也帮我们配好了

```java
matchIfMissing = true //这个的意思就是如果没有匹配成功的话这个字段默认为true
```

并且这个组件是@ConditionalOnMissingBean ，意思就是如果我们没有这个类型的组件才会帮我们配置。

也就是说我们自己也可以在Config 配置文件中添加一个这个类型的组件，如果我们自己配置了就不会添加这个了

![image-20220312155726862](https://image.imxyu.cn/file/image-20220312155726862.png)

可以看到这个也是通过properties 进行赋值的，绑定了ServerProperties.class

也就是说我们写有关server前缀的注解就会被封装到这个类上

![image-20220312160615376](https://image.imxyu.cn/file/image-20220312160615376.png)

总结：

- SpringBoot先加载所有的自动配置类  xxxxxAutoConfiguration
- 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。xxxxProperties里面拿。xxxProperties和配置文件进行了绑定
- 生效的配置类就会给容器中装配很多组件
- 只要容器中有这些组件，相当于这些功能就有了
- 定制化配置

- - 用户直接自己@Bean替换底层的组件
  - 用户去看这个组件是获取的配置文件什么值就去修改。（可以直接看底层源码绑定的什么配置文件中的前缀值，也可以直接看官网的介绍https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties）

**xxxxxAutoConfiguration ---> 组件  --->** **xxxxProperties里面拿值  ----> application.properties**

## 3.4、最佳实践

- 引入场景依赖

- - https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter

- 查看自动配置了哪些（选做）

- - 自己分析，引入场景对应的自动配置一般都生效了
  - 在application.properties配置文件中debug=true开启自动配置报告。Negative（不生效）\Positive（生效）

- 是否需要修改

- - 参照文档修改配置项

- - - https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties
    - 自己分析。xxxxProperties绑定了配置文件的哪些。

- - 自定义加入或者替换组件

- - - @Bean、@Component。。。

- - 自定义器  **XXXXXCustomizer**；
  - ......

# 4、开发小技巧

## 4.1、Lombok

简化JavaBean开发

```java
//1. 添加dep  
<dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>


//2.idea中搜索安装lombok插件
```



```java
===============================简化JavaBean开发===================================
@NoArgsConstructor
//@AllArgsConstructor
@Data
@ToString
@EqualsAndHashCode
public class User {

    private String name;
    private Integer age;

    private Pet pet;

    public User(String name,Integer age){
        this.name = name;
        this.age = age;
    }


}



================================简化日志开发===================================
@Slf4j
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String handle01(@RequestParam("name") String name){
        
        log.info("请求进来了....");
        
        return "Hello, Spring Boot 2!"+"你好："+name;
    }
}
```

> 会在**编译完**生成相应的方法

## 4.2、dev-tools 热更新



```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
```

项目或者页面修改以后：Ctrl+F9；

> 这个只会重启我们的项目，省去了我们手动重启的操作，如果想使用更纯正的热更新使用付费的jrebal

4.3、Spring Initailizr（项目初始化向导

## 4.3、Spring Initailizr（项目初始化向导）

### 0、选择我们需要的开发场景

![image.png](https://image.imxyu.cn/file/1602922147241-73fb2496-e795-4b5a-b909-a18c6011a028.png)

### 1、自动依赖引入

![image.png](https://image.imxyu.cn/file/1602921777330-8fc5c198-75da-4ff9-b82c-71ee3fe18af8.png)

### 2、自动创建项目结构

会自动帮我们创建好这些文件，对应的静态资源和页面都放入这个文件夹下就可以了

![image.png](https://cdn.nlark.com/yuque/0/2020/png/1354552/1602921758313-5099fe18-4c7b-4417-bf6f-2f40b9028296.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_18%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



### 3、自动编写好主配置类

![image.png](https://image.imxyu.cn/file/1602922039074-79e98aad-8158-4113-a7e7-305b57b0a6bf.png)



# 注解总结

```java
创建对象的：
@Controller: 放在类的上面，创建控制器对象，注入到容器中
@RestController: 放在类的上面，创建控制器对象，注入到容器中。
             作用：复合注解是@Controller , @ResponseBody, 使用这个注解类的，里面的控制器方法的返回值都是数据

@Service ： 放在业务层的实现类上面，创建service对象，注入到容器
@Repository : 放在dao层的实现类上面，创建dao对象，放入到容器。 没有使用这个注解，是因为现在使用MyBatis框架，  dao对象是MyBatis通过代理生成的。 不需要使用@Repository、 所以没有使用。
@Component:  放在类的上面，创建此类的对象，放入到容器中。 

赋值的：
@Value ： 简单类型的赋值， 例如 在属性的上面使用@Value("李四") private String name
          还可以使用@Value,获取配置文件者的数据（properties或yml）。 
          @Value("${server.port}") private Integer port

@Autowired: 引用类型赋值自动注入的，支持byName, byType. 默认是byType 。 放在属性的上面，也可以放在构造方法的上面。 推荐是放在构造方法的上面
@Qualifer:  给引用类型赋值，使用byName方式。   
            @Autowird, @Qualifer都是Spring框架提供的。

@Resource ： 来自jdk中的定义， javax.annotation。 实现引用类型的自动注入， 支持byName, byType.
             默认是byName, 如果byName失败， 再使用byType注入。 在属性上面使用


其他：
@Configuration ： 放在类的上面，表示这是个配置类，相当于xml配置文件

@Bean：放在方法的上面， 把方法的返回值对象，注入到spring容器中。

@ImportResource ： 加载其他的xml配置文件， 把文件中的对象注入到spring容器中

@PropertySource ： 读取其他的properties属性配置文件

@ComponentScan： 扫描器 ，指定包名，扫描注解的

@ResponseBody: 放在方法的上面，表示方法的返回值是数据， 不是视图
@RequestBody : 把请求体中的数据，读取出来， 转为java对象使用。

@ControllerAdvice:  控制器增强， 放在类的上面， 表示此类提供了方法，可以对controller增强功能。

@ExceptionHandler : 处理异常的，放在方法的上面

@Transcational :  处理事务的， 放在service实现类的public方法上面， 表示此方法有事务


SpringBoot中使用的注解
    
@SpringBootApplication ： 放在启动类上面， 包含了@SpringBootConfiguration
                          @EnableAutoConfiguration， @ComponentScan


    
MyBatis相关的注解

@Mapper ： 放在类的上面 ， 让MyBatis找到接口， 创建他的代理对象    
@MapperScan :放在主类的上面 ， 指定扫描的包， 把这个包中的所有接口都创建代理对象。 对象注入到容器中
@Param ： 放在dao接口的方法的形参前面， 作为命名参数使用的。
    
Dubbo注解
@DubboService: 在提供者端使用的，暴露服务的， 放在接口的实现类上面
@DubboReference:  在消费者端使用的， 引用远程服务， 放在属性上面使用。
@EnableDubbo : 放在主类上面， 表示当前引用启用Dubbo功能。
    
    
```

