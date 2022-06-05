使用gradle ，创建一个普通的web项目

![image-20220208210852730](https://image.imxyu.cn/file/image-20220208210852730.png)

在gradle中导入spring-webmvc 的依赖

![image-20220208210944026](https://image.imxyu.cn/file/image-20220208210944026.png)



参考官方文档中spring-mvc 的构建，这里使用注解版的

```java
/**
 * 只要写了这个，相当于配置了SpringMVC的DispatcherServlet
 * 1、Tomcat一启动就加载他
 * 		1）、创建了容器、制定了配置类（所有ioc、aop等spring的功能就ok）
 * 		2）、注册一个Servlet；	DispatcherServlet；
 * 		3）、以后所有的请求都交给了 DispatcherServlet；
 * 	效果，访问Tomcat部署的这个Web应用下的所有请求都会被 	DispatcherServlet 处理
 * 	DispatcherServlet就会进入强大的基于注解的mvc处理流程（@GetMapping）
 * 必须Servlet3.0以上才可以；Tomcat6.0以上才支持Servlet3.0规范
 *
 * Servlet3.0是javaEE的Web的规范标准，Tomcat是Servlet3.0规范的一个实现；
 */
public class AppStarter implements WebApplicationInitializer {
//	@Override
	public void onStartup(ServletContext servletContext) throws ServletException {
		//1、创建ioc容器
		AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
		context.register(SpringConfig.class); //2、传入一个配置类
		//以上截止，ioc容器都没有启动
		//3、配置了 DispatcherServlet,利用Servlet的初始化机制
		DispatcherServlet servlet = new DispatcherServlet(context);
		ServletRegistration.Dynamic registration = servletContext.addServlet("app", servlet);
		registration.setLoadOnStartup(1);
		registration.addMapping("/"); //映射路径

		//启动了容器？上面的Servlet添加到 servletContext 里面以后，Tomcat就会对 DispatcherServlet进行初始化
		//<servlet></servlet>
//		servletContext.addServlet("abc",XXXX.class)

	}
}
```

配置类，包扫描

扫描所有的controller

```java
/**
 * SpringMVC只扫描controller组件，可以不指定父容器类，让MVC扫所有。@Component+@RequestMapping就生效了
 */
@ComponentScan(value = "com.atguigu.web",includeFilters = {
		@ComponentScan.Filter(type= FilterType.ANNOTATION,value = Controller.class)
},useDefaultFilters = false)
public class SpringMVCConfig {
	//SpringMVC的子容器，能扫描的Spring容器中的组件


}
```



这里有个疑问，我们在AppStarter类上没有标注@Component接口，tomcat是怎么知道一启动就要加载这个类的？

> 这里我们说一下Tomcat和servlet的关系
>
> servlet 3.0 是javaEE 的Web的规范标准，Tomcat 是servlet 3.0规范的实现 。因为实现了servlet 规范所以Tomcat中能运行servlet程序，Tomcat就是servlet的容器

我们带着这个问题，去看下一节
