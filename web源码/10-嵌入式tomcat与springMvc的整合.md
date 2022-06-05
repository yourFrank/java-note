## 手动创建tomcat和servlet

明白了springMvc的原理，我们完全可以手写一个tomcat来启动servlet来处理我们的请求

首先我们直接创建一个maven项目

引入tomcat和web-mvc的包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>springboot-first</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.5</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.tomcat.embed/tomcat-embed-core -->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>8.5.64</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.tomcat.embed/tomcat-embed-jasper -->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <version>8.5.64</version>
        </dependency>


    </dependencies>

</project>
```



```java
public class Main {

    public static void main(String[] args) throws LifecycleException {
        //自己写Tomcat的启动源码
        Tomcat tomcat = new Tomcat();

		//指定端口
        tomcat.setPort(8888);
        tomcat.setHostname("localhost");
        //指定tomcat根路径
        tomcat.setBaseDir(".");

        //指定tomcat项目路径
        Context context = tomcat.addWebapp("/boot", System.getProperty("user.dir") + "/src/main");

        //给Tomcat里面添加一个Servlet
        Wrapper hello = tomcat.addServlet("/boot", "hello", new HelloServlet());

        hello.addMapping("/66"); //指定处理的请求


        tomcat.start();//启动tomcat 注解版MVC利用Tomcat SPI机制


        tomcat.getServer().await(); //服务器等待

    }

}

```

编写一个servlet来处理我们的请求

```java
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("hello tomcat");
    }
}

```

![image-20220304201657667](https://image.imxyu.cn/file/image-20220304201657667.png)

同样的我们可以直接创建DispatcherServlet，然后加入到tomcat中，让它帮我们处理所有请求

```java
public class Main {

    public static void main(String[] args) throws LifecycleException {
        //自己写Tomcat的启动源码
        Tomcat tomcat = new Tomcat();

        tomcat.setPort(8888);
        tomcat.setHostname("localhost");
        tomcat.setBaseDir(".");

        Context context = tomcat.addWebapp("/boot", System.getProperty("user.dir") + "/src/main");
		//手动new DispatcherServlet加入到tomcat中
        DispatcherServlet servlet = new DispatcherServlet();
        //给Tomcat里面添加一个Servlet
//        Wrapper hello = tomcat.addServlet("/boot", "hello", new HelloServlet());
        Wrapper hello = tomcat.addServlet("/boot", "hello", servlet);
        hello.addMapping("/"); //指定处理的请求,让DispatcherServlet处理所有请求

        //自己创建 DispatcherServlet 对象，并且创建ioc容器，DispatcherServlet里面有ioc容器

        tomcat.start();//启动tomcat 注解版MVC利用Tomcat SPI机制


        tomcat.getServer().await(); //服务器等待

    }

}
```



## 创建tomcat利用SPI机制加载servlet

当然我们这样编写servlet启动比较麻烦，我们可以直接使用tomcat 的SPI机制去加载父子容器（我们之前说过）

其他代码都不要了

```java
/**
 * 怎么启动起来？
 * Tomcat启动
 * SPI机制下 QuickAppStarter生效创建 ioc容器配置DispatcherServlet等各种组件
 *
 * 导入各种starter依赖，SpringBoot封装了很多的自动配置，帮我们给容器中放了很多组件。
 * SpringBoot封装了功能的自动配置
 *
 * WebServerFactory做到了
 */
public class Main {

    public static void main(String[] args) throws LifecycleException {
        //自己写Tomcat的启动源码
        Tomcat tomcat = new Tomcat();


        tomcat.setPort(8888);
        tomcat.setHostname("localhost");
        tomcat.setBaseDir(".");

        Context context = tomcat.addWebapp("/boot", System.getProperty("user.dir") + "/src/main");

        tomcat.start();//启动tomcat 注解版MVC利用Tomcat SPI机制

        tomcat.getServer().await(); //服务器等待

    }

}

```

我们直接把之前的所有类都拿过来，这是拿过来后的目录结构

![image-20220304202534790](https://image.imxyu.cn/file/image-20220304202534790.png)

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
//    super.customizeRegistration(registration);

//    registration.addMapping("");//
   }
}
```



```java
/**
 * Spring不扫描controller组件、AOP咋实现的....
 */
@ComponentScan(value = "com.atguigu.boot",excludeFilters = {
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
@ComponentScan(value = "com.atguigu.boot",includeFilters = {
      @ComponentScan.Filter(type= FilterType.ANNOTATION,value = Controller.class)
},useDefaultFilters = false)
public class SpringMVCConfig {
   //SpringMVC的子容器，能扫描的Spring容器中的组件


}
```

只需要一个类QuickAppStarter extends AbstractAnnotationConfigDispatcherServletInitializer和

和两个容器的配置文件就行，这样tomcat启动之后就会利用SPI机制创建容器

然后我们创建一个controller

```java
@RestController
public class HelloController {

    @GetMapping("/hello66")
    public String hello(){

        return "66666666~~~~~";
    }
}
```

可以看到容器启动后两个容器都已经创建并且初始化了

![image-20220304203000625](https://image.imxyu.cn/file/image-20220304203000625.png)

我们直接访问页面就能看到了~

![image-20220304202847587](https://image.imxyu.cn/file/image-20220304202847587.png)

> 上面这个Main 其实就是Springboot启动类要干的，srpingboot其实也就是在某一时刻帮我们启动了一个tomcat, 当然没我们写的这么简单。springboot不仅允许嵌入式的tomcat还可以非嵌入式的，这些都是WebServerFactory做到的。后期springboot再说
>
> ```
> 允许通过导入各种starter依赖，SpringBoot封装了很多的自动配置，帮我们给容器中放了很多组件。
> SpringBoot封装了功能的自动配置仅此而已
> 其他的都是继承了spring 的ioc\aop 和springmvc中的web功能
> ```

