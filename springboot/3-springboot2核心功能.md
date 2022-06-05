视频地址：https://www.bilibili.com/video/BV19K4y1L7MT?from=search&seid=5309862794027495419&spm_id_from=333.337.0.0

雷丰阳老师语雀笔记地址：https://www.yuque.com/atguigu/springboot/na3pfd

# Web 功能

## 错误异常处理

### 1、默认规则

- 默认情况下，Spring Boot提供`/error`处理所有错误的映射
- 对于机器客户端，它将生成JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。对于浏览器客户端，响应一个“ whitelabel”错误视图，以HTML格式呈现相同的数据

postman ：

- ![img](https://image.imxyu.cn/file/1606024421363-77083c34-0b0e-4698-bb72-42da351d3944.png)

浏览器：

![image.png](https://image.imxyu.cn/file/1606024616835-bc491bf0-c3b1-4ac3-b886-d4ff3c9874ce.png)

- **要对其进行自定义，添加**`View`**解析为**`error`
- 要完全替换默认行为，可以实现 `ErrorController `并注册该类型的Bean定义，或添加`ErrorAttributes类型的组件`以使用现有机制但替换其内容。
- error/下的4xx，5xx页面会被自动解析；

![image.png](https://image.imxyu.cn/file/1606024592756-d4ab8a6b-ec37-426b-8b39-010463603d57.png)

### 2、默认错误处理原理【源码】

#### 异常处理组件

主要分析这个自动配置类中给我们添加了什么组件

![image-20220314170637959](https://image.imxyu.cn/file/image-20220314170637959.png)



- **ErrorMvcAutoConfiguration  自动配置异常处理规则**

- - **容器中的组件：类型：DefaultErrorAttributes ->** **id：errorAttributes**

- - - **public class** **DefaultErrorAttributes** **implements** **ErrorAttributes**, **HandlerExceptionResolver**
    - **DefaultErrorAttributes**：定义错误页面中可以包含哪些数据。
    - ![img](https://image.imxyu.cn/file/1606044430037-8d599e30-1679-407c-96b7-4df345848fa4.png)
    - ![img](https://image.imxyu.cn/file/1606044487738-8cb1dcda-08c5-4104-a634-b2468512e60f.png)

- - 容器中的组件：类型：**BasicErrorController** --> **id：basicErrorController**（json+白页 适配响应）

- - - **处理默认** **/error 路径的请求；页面响应** **new** ModelAndView(**"error"**, model)；
    - **容器中有组件 View**->**id是error**；（响应默认错误页）
    - 容器中放组件 **BeanNameViewResolver（视图解析器）；按照返回的视图名作为组件的id去容器中找View对象。**

- - **容器中的组件：**类型：**DefaultErrorViewResolver -> id：**conventionErrorViewResolver

- - - 如果发生错误，会以HTTP的状态码 作为视图页地址（viewName），找到真正的页面
    - error/404、5xx.html



如果想要返回页面；就会找error视图【**StaticView**】。(默认是一个白页)





下面我们看看放到容器中的这几个组件的源码

**BasicErrorController** 首先来看这个，会根据不同的mediaType 类型进行不同的相应（直接写出json 或者是白页面（就是跳转view 页渲染出来的））

如果是浏览器会把转到view 名字为eroor 的视图页面

![image-20220314171546213](https://image.imxyu.cn/file/image-20220314171546213.png) 

写出去json

![image-20220314171711317](https://image.imxyu.cn/file/image-20220314171711317.png)



**id 为error的view组件**

![image-20220314173326879](https://image.imxyu.cn/file/image-20220314173326879.png)

render 方法为写出html 页面

![image-20220314173400111](https://image.imxyu.cn/file/image-20220314173400111.png)

> 这就是上面的id：basicErrorController 返回modelAndView 中 error 的视图，就会通过它处理







**DefaultErrorViewResolver** 

viewResolver 一听名字**Resolver**就是处理去生成相应的view对象，可以看到这里直接根据httpStatus 直接将viewName 设置成了error/+对应的状态码，

因此就可以知道为什么我们可以在error/中放404, 5xx 这些文件了

![image-20220314172112557](https://image.imxyu.cn/file/image-20220314172112557.png)



**BeanNameViewResolver** ：按照返回的视图名，去找对应id 的view对象

![image-20220314173111755](https://image.imxyu.cn/file/image-20220314173111755.png)

> 上面的id：basicErrorController 找名字error 的视图，就会通过这个解析器找到容器中id 为error 的视图



DefaultErrorAttributes 定义了错误页面能放的属性

![image-20220314183814096](https://image.imxyu.cn/file/image-20220314183814096.png)



#### 异常处理流程

首先我们模拟一个异常，看一下这个异常是怎么处理的，制造一个数学运算异常

![image-20220314185745312](https://image.imxyu.cn/file/image-20220314185745312.png)



到我们的dispatcherServelt 中看一下反射调用方法的时候，可以看到在反射调用方法的时候返回了数学运算异常

然后在catch中把这个异常保存在disptachException 中，继续走下面视图渲染的逻辑

![image-20220314190040987](https://image.imxyu.cn/file/image-20220314190040987.png)

在视图渲染的前半部分会进行异常的处理

![image-20220314190135288](https://image.imxyu.cn/file/image-20220314190135288.png)

在这个异常处理中，会调用系统中自带的这几个异常处理器进行处理异常，看这几个异常处理器哪个能处理

![image-20220314190208216](https://image.imxyu.cn/file/image-20220314190208216.png)

结果这所有的异常处理器都处理不了，然后就抛出去了

![image-20220314190420121](https://image.imxyu.cn/file/image-20220314190420121.png)



我们直接放行断点，发现又进来了一次请求。这个请求就是调用的tomcat的response.sendError 机制发的 /error的请求

![image-20220314190819274](https://image.imxyu.cn/file/image-20220314190819274.png)

而这个error 请求底层就会走的这个controller

![image-20220314191040490](https://image.imxyu.cn/file/image-20220314191040490.png)

可以看到这里直接通过调用resolveErrorView（），解析获得相应的view

```
ModelAndView modelAndView = resolveErrorView(request, response, status, model);
```

![image-20220314191134435](https://image.imxyu.cn/file/image-20220314191134435.png)

这个就是spring给我们添加的errorViewResolvers

![image-20220314191244760](https://image.imxyu.cn/file/image-20220314191244760.png)

处理的过程就是直接拿到reuqest 中的错误码，返回了一个error/+错误码的 view

这个view 最终会被themeleaf 解析，然后到相应的页面

![image-20220314191343508](https://image.imxyu.cn/file/image-20220314191343508.png)



![image-20220314191510652](https://image.imxyu.cn/file/image-20220314191510652.png)



**总结**：

1、执行目标方法，目标方法运行期间有任何异常都会被catch、而且标志当前请求结束；并且用 **dispatchException** 

2、进入视图解析流程（页面渲染？） 

processDispatchResult(processedRequest, response, mappedHandler, **mv**, **dispatchException**);

3、**mv** = **processHandlerException**；处理handler发生的异常，处理完成返回ModelAndView；

- 1、遍历所有的 **handlerExceptionResolvers，看谁能处理当前异常【****HandlerExceptionResolver处理器异常解析器****】**
- ![img](https://image.imxyu.cn/file/1606047252166-ce71c3a1-0e0e-4499-90f4-6d80014ca19f.png)
- **2、系统默认的  异常解析器；**
- ![img](https://image.imxyu.cn/file/1606047109161-c68a46c1-202a-4db1-bbeb-23fcae49bbe9.png)

- - **1、DefaultErrorAttributes先来处理异常。把异常信息保存到rrequest域，并且返回null；**
  - **2、默认没有任何人能处理异常，所以异常会被抛出**

- - - **1、如果没有任何人能处理最终底层就会发送 /error 请求。会被底层的BasicErrorController处理**
    - **2、解析错误视图；遍历所有的**  **ErrorViewResolver  看谁能解析。**
    - ![img](https://image.imxyu.cn/file/1606047900473-e31c1dc3-7a5f-4f70-97de-5203429781fa.png)
    - **3、默认的** **DefaultErrorViewResolver ,作用是把响应状态码作为错误页的地址，error/500.html** 
    - **4、模板引擎最终响应这个页面** **error/500.html** 



### 3、定制异常处理的几种方式

#### 1、在error/ 中自定义错误页

上面我们说过可以通过自定义错误页的方式，底层默认的错误解析器会找到我们的错误页

- error/404.html   error/5xx.html；有精确的错误状态码页面就匹配精确，没有就找 4xx.html；如果都没有就触发白页

![image-20220314192142281](https://image.imxyu.cn/file/image-20220314192142281.png)

我们可以取到相应的错误信息，用一个错误页面4xx 处理400 和404 的问题

![image-20220314192156345](https://image.imxyu.cn/file/image-20220314192156345.png)





#### 2、@ControllerAdvice+@ExceptionHandler

第二种我们可以通过这两个注解来自定义异常的处理逻辑

因为上面我们看源码的时候看了异常处理的逻辑最后也是要返回一个modelAndView 的，我们这里直接返回login页面（因为有thymeleaf模板引擎，会帮我们拼接上前缀和后缀，最后直接找到templates/ login.html）

这个@ControllerAdvice注解能帮我们这个类注册到容器中

![image-20220314193922113](https://image.imxyu.cn/file/image-20220314193922113.png)

然后通过@ExceptionHandler 在方法上标注就可以

我们让发生这个异常的情况下返回我们的登录页面

```java
/**
 * 处理整个web controller的异常
 */
@Slf4j
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler({ArithmeticException.class,NullPointerException.class})  //处理异常
    public String handleArithException(Exception e){

        log.error("异常是：{}",e);
        return "login"; //视图地址
    }
}

```





这个的原理就是我们上面说的在for使用异常处理器进行处理的过程中，调用的ExceptionHandLExceptionResolver



![image-20220314190208216](https://image.imxyu.cn/file/image-20220314190208216.png)

这个异常处理器会找到所有的@ExceptionHandler 注解的方法，看哪个能处理这个异常

![image-20220314193445335](https://image.imxyu.cn/file/image-20220314193445335.png)

判断到我们的这个能处理这个异常，就使用我们的这个异常处理，

![image-20220314193524168](https://image.imxyu.cn/file/image-20220314193524168.png)

找到之后返回modelAndView 类型。如果返回的不为空的话，就像正常的一样，走视图渲染的流程

![image-20220314193822770](https://image.imxyu.cn/file/image-20220314193822770.png)

![image-20220314193811194](https://image.imxyu.cn/file/image-20220314193811194.png)



#### 3、@ResponseStatus+自定义异常

我们可以利用抛出一个自定义异常

![image-20220314195948765](https://image.imxyu.cn/file/image-20220314195948765.png)



这个就是我们自定义的异常，我们在这个类上标注了@ResponseStatus 这个注解指明了状态码和错误的原因

```java
@ResponseStatus(value= HttpStatus.FORBIDDEN,reason = "用户数量太多")
public class UserTooManyException extends RuntimeException {

    public  UserTooManyException(){

    }
    public  UserTooManyException(String message){
        super(message);
    }
}

```

底层会通过第二个ResponseStatusExceptionResolver  去处理这个我们标注了@ResponseStatus注解的异常

![image-20220314190208216](https://image.imxyu.cn/file/image-20220314190208216.png)

我们可以直接继承这个类型的异常，也可以直接标注这个注解。都可以帮我们处理

![image-20220314200243660](https://image.imxyu.cn/file/image-20220314200243660.png)

处理的流程就是拿到我们定义的状态码和错误原因

![image-20220314200311898](https://image.imxyu.cn/file/image-20220314200311898.png)

看好这个如果我们的原因不为空，直接调用tomcat 的sendError（自定义的状态码）进行处理

![image-20220314200352671](https://image.imxyu.cn/file/image-20220314200352671.png)



因为上面说了底层有一个ErrorController 去处理这个error 的请求，会直接通过viewResolver 然后去找到我们在tempaltes 中放的页面

#### 小插曲DefaultHandlerExceptionResolver

最后还有一个这个异常处理器没说，这里一块说了

![image-20220314190208216](https://image.imxyu.cn/file/image-20220314190208216.png)

这个异常处理器就是为了处理spring底层的异常的，如果出现了参数缺失400等 spring自己的异常就会使用这个

![image-20220314200722409](https://image.imxyu.cn/file/image-20220314200722409.png)

它的处理逻辑也是直接发送sendError

![image-20220314200844117](https://image.imxyu.cn/file/image-20220314200844117.png)

> 也就是说如果我们在controller 处理的过程中，我们也可以直接调用这个response.sendError
>
> 也会走默认的那个controller, 走到我们自定义的页面



#### 4、自定义异常解析器

上面这几种都是实现了handlerExceptionResolver 接口的

如果我们不满意这些异常解析器，我们也可以自己定义异常解析器

![image-20220314201108225](https://image.imxyu.cn/file/image-20220314201108225.png)



```java
@Order(value= Ordered.HIGHEST_PRECEDENCE)  //优先级，数字越小优先级越高
@Component
public class CustomerHandlerExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request,
                                         HttpServletResponse response,
                                         Object handler, Exception ex) {

        try {
            response.sendError(511,"我喜欢的错误");
        } catch (IOException e) {
            e.printStackTrace();
        }
        return new ModelAndView();
    }
}
```

这样底层就有了我们的异常处理器，这里需要注意的是如果我们想要我们的作为默认的异常解析器需要在注解上标注@Order(value= Ordered.HIGHEST_PRECEDENCE)  //优先级，数字越小优先级越高，否则的话我们的异常 解析器会在最后一个，会让系统的这些都处理完了才到我们的

![image.png](https://image.imxyu.cn/file/1606114688649-e6502134-88b3-48db-a463-04c23eddedc7.png)





#### 总结：

- 自定义错误页

- - error/404.html   error/5xx.html；有精确的错误状态码页面就匹配精确，没有就找 4xx.html；如果都没有就触发白页

- @ControllerAdvice+@ExceptionHandler处理全局异常；底层是 **ExceptionHandlerExceptionResolver 支持的**
- @ResponseStatus+自定义异常 ；底层是 **ResponseStatusExceptionResolver ，把responsestatus注解的信息底层调用** **response.sendError(statusCode, resolvedReason)；tomcat发送的/error**
- Spring底层的异常，如 参数类型转换异常；**DefaultHandlerExceptionResolver 处理框架底层的异常。**

- - response.sendError(HttpServletResponse.**SC_BAD_REQUEST**, ex.getMessage()); 
  - ![img](https://image.imxyu.cn/file/1606114118010-f4aaf5ee-2747-4402-bc82-08321b2490ed.png)

- 自定义实现 HandlerExceptionResolver 处理异常；可以作为默认的全局异常处理规则

- - ![img](https://image.imxyu.cn/file/1606114688649-e6502134-88b3-48db-a463-04c23eddedc7.png)

- **ErrorViewResolver**  实现自定义处理异常；

- - response.sendError 。error请求就会转给controller
  - 你的异常没有任何人能处理。tomcat底层 response.sendError。error请求就会转给controller
  - **basicErrorController 要去的页面地址是** **ErrorViewResolver**  ；



## Web原生组件注入（Servlet、Filter、Listener）

### 1、使用Servlet API（推荐）

@ServletComponentScan(basePackages = **"com.atguigu.admin"**) :指定原生Servlet组件都放在那里

@WebServlet(urlPatterns = **"/my"**)：效果：我们访问/my 直接响应，**没有经过Spring的拦截器（因为我们之前配置了拦截所有的拦截器）？**

@WebFilter(urlPatterns={**"/css/\*"**,**"/images/\*"**})

@WebListener

```java
@Slf4j
@WebFilter(urlPatterns={"/css/*","/images/*"}) //my
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("MyFilter初始化完成");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("MyFilter工作");
        chain.doFilter(request,response);
    }

    @Override
    public void destroy() {
        log.info("MyFilter销毁");
    }
}

```



```java

@WebServlet(urlPatterns = "/my")
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("66666");
    }
}

```



```java

@Slf4j
@WebListener
public class MySwervletContextListener implements ServletContextListener {


    @Override
    public void contextInitialized(ServletContextEvent sce) {
        log.info("MySwervletContextListener监听到项目初始化完成");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        log.info("MySwervletContextListener监听到项目销毁");
    }
}

```

定义完之后扫描对应包即可

![image-20220314203009413](https://image.imxyu.cn/file/image-20220314203009413.png)



> 注意上面我们自定义的servlet 不会经过我们spring的拦截器interceptor ，那么为什么呢 ？ 我们来看一下、

首先我们给容器中注册了一个disptacherServlet 组件

但是此时注意，这个dispatcherServlet是不会生效的，只有放到tomcat 中才会生效。我们继续往下看 

![image-20220314203814299](https://image.imxyu.cn/file/image-20220314203814299.png)



然后通过DispatcherServletRegistrationBean 来注册到容器中的，也就是我们下面写的第二种方式注册的

![image-20220314203922083](https://image.imxyu.cn/file/image-20220314203922083.png)



并且我们看到这里传入的路径默认是 / , 也就是说disptacherServlet 默认会处理 / 路径 

![image-20220314204103417](https://image.imxyu.cn/file/image-20220314204103417.png)

并且绑定在这个配置文件中，因此我们可以通过配置文件修改它能处理的路径

![image-20220314204152274](https://image.imxyu.cn/file/image-20220314204152274.png)



![image-20220314204221800](https://image.imxyu.cn/file/image-20220314204221800.png)



目前tomcat 中有两个Servlet，我们知道tomcat 会通过精确匹配的原则，也就是说现在是下面这种情况，

当我们访问/my 的时候，tomcat会使用MyServlet 这个，因此不会走我们的dispatcherServlet , 因此不会走spring的流程

Tomcat-Servlet；

多个Servlet都能处理到同一层路径，精确优选原则

A： /my/

B： /my/1



![image.png](https://image.imxyu.cn/file/1606284869220-8b63d54b-39c4-40f6-b226-f5f095ef9304.png)

总结：

扩展：DispatchServlet 如何注册进来

- 容器中自动配置了  DispatcherServlet  属性绑定到 WebMvcProperties；对应的配置文件配置项是 **spring.mvc。**
- **通过** **ServletRegistrationBean**<DispatcherServlet> 把 DispatcherServlet  配置进来。
- 默认映射的是 / 路径。

### 2、使用RegistrationBean

```
ServletRegistrationBean`, `FilterRegistrationBean`, and `ServletListenerRegistrationBean
```

```java
@Configuration
public class MyRegistConfig {

    @Bean
    public ServletRegistrationBean myServlet(){
        MyServlet myServlet = new MyServlet();

        return new ServletRegistrationBean(myServlet,"/my","/my02");
    }


    @Bean
    public FilterRegistrationBean myFilter(){

        MyFilter myFilter = new MyFilter();
//        return new FilterRegistrationBean(myFilter,myServlet());
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(myFilter);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/my","/css/*"));
        return filterRegistrationBean;
    }

    @Bean
    public ServletListenerRegistrationBean myListener(){
        MySwervletContextListener mySwervletContextListener = new MySwervletContextListener();
        return new ServletListenerRegistrationBean(mySwervletContextListener);
    }
}
```



## 嵌入式服务器



### 1、切换嵌入式Servlet容器

- 默认支持的webServer

- - `Tomcat`, `Jetty`, or `Undertow`
  - `ServletWebServerApplicationContext 容器启动寻找ServletWebServerFactory 并引导创建服务器`

- 切换服务器(我们需要把web 启动器中帮我们引入的tomcat 依赖去除，然后引入我们需要的服务器的jar包，这里以unsertow为例)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-undertow</artifactId>
 </dependency>
```

![image-20220314210904482](https://image.imxyu.cn/file/image-20220314210904482.png)

底层有一个这个servletWebServer的自动配置类

![image-20220314205405284](https://image.imxyu.cn/file/image-20220314205405284.png)



这个配置类会根据项目当前有什么jar包会帮我们创建对应的工厂

此时我们引入了web 的，就会有TomcatServletWebServerFactory

![image-20220314205614690](https://image.imxyu.cn/file/image-20220314205614690.png)



ServletWebServerApplicationContext 这个是我们的ioc 容器

而当我们项目启动的时候会调用ServletWebServerApplicationContext 类的OnRefresh（） 方法，会创建webServer

> 这里具体是怎么调用ioc 容器刷新的可以看大厂学院的笔记

![image-20220314205855957](https://image.imxyu.cn/file/image-20220314205855957.png)

我们debug来看一下这个创建Web服务器的流程

![image-20220314210651995](https://image.imxyu.cn/file/image-20220314210651995.png)



调用获取web 服务器工厂，这里是根据类型获取的ServletWebServerFactory.class ，而我们上面自动配置类给容器中放了TomcatServletWebServerFactory, 这里就会找到我们的TomcatServletWebServerFactory

![image-20220314210644781](https://image.imxyu.cn/file/image-20220314210644781.png)

然后调用TomcatServletWebServerFactory 工厂得到对应的tomcat 服务器，我们点进来看一下

![image-20220314211042320](https://image.imxyu.cn/file/image-20220314211042320.png)

这里就是创建tomcat 服务器的过程，创建完了之后会调用getTomcatWebServer(tomcat);

![image-20220314211349290](https://image.imxyu.cn/file/image-20220314211349290.png)



![image-20220314211403364](https://image.imxyu.cn/file/image-20220314211403364.png)



![image-20220314211411192](https://image.imxyu.cn/file/image-20220314211411192.png)

在初始化的方法中有一个tomcat.start() 的方法，服务器启动

![image-20220314211418729](https://image.imxyu.cn/file/image-20220314211418729.png)



> 因此如果我们换一个服务器的话，此时就会拿到另一个服务器的工厂创建对应的服务器启动



- 原理

- - SpringBoot应用启动发现当前是Web应用（我们引入了web的依赖）。web场景包-导入tomcat容器
  - web应用会创建一个web版的ioc容器 `ServletWebServerApplicationContext` 
  - `ServletWebServerApplicationContext` 启动的时候寻找 `**ServletWebServerFactory**``（Servlet 的web服务器工厂---> Servlet 的web服务器）` 
  - SpringBoot底层默认有很多的WebServer工厂；`TomcatServletWebServerFactory`, `JettyServletWebServerFactory`, or `UndertowServletWebServerFactory`
  - `底层直接会有一个自动配置类。ServletWebServerFactoryAutoConfiguration`
  - `ServletWebServerFactoryAutoConfiguration导入了ServletWebServerFactoryConfiguration（配置类）`
  - `ServletWebServerFactoryConfiguration 配置类 根据动态判断系统中到底导入了那个Web服务器的包。（默认是web-starter导入tomcat包），容器中就有 TomcatServletWebServerFactory`
  - `TomcatServletWebServerFactory 创建出Tomcat服务器并启动；TomcatWebServer 的构造器拥有初始化方法initialize---this.tomcat.start();`
  - `内嵌服务器，就是手动把启动服务器的代码调用（tomcat核心jar包存在）`

### 2、定制Servlet容器

可以根据自动配置类看到绑定的属性是通过server开头的

![image-20220314212116280](https://image.imxyu.cn/file/image-20220314212116280.png)



![image-20220314212323425](https://image.imxyu.cn/file/image-20220314212323425.png)

这个配置文件中，包括tomcat 最后都会从这个文件中取值，然后设置进去。因此我们只需要修改配置文件中的server.就可以了

![image-20220314212359479](https://image.imxyu.cn/file/image-20220314212359479.png)



- 实现  **WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>**   定制化器

- - 把配置文件的值和`**ServletWebServerFactory 进行绑定**`

- 修改配置文件 **server.xxx**（这个只能修改容器的属性比如端口号什么的，不能自定义容器）

- 直接自定义 **ConfigurableServletWebServerFactory** （因为我们上面说过都是找的对应的ServeletWebServerFactory创建的容器对象）

  我们可以直接用它的子类，里面会提供了很多方法。使用这个来自定义一个ConfigurableServletWebServerFactory 注册到容器中，到时候就会找我们的

![image-20220314212719533](https://image.imxyu.cn/file/image-20220314212719533.png)

**xxxxxCustomizer：定制化器，可以改变xxxx的默认规则**

```java
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

    @Override
    public void customize(ConfigurableServletWebServerFactory server) {
        server.setPort(9000);
    }

}
```



## 定制原理

### 定制化的常见方式 

- 修改配置文件

- `xxxxxCustomizer`

- 编写自定义的配置类  `xxxConfiguration` + `@Bean`替换、增加容器中默认组件，视图解析器

- Web应用 编写一个配置类实现 `WebMvcConfigurer` 即可定制化web功能 + `@Bean`给容器中再扩展一些组件

```java
@Configuration
public class AdminWebConfig implements WebMvcConfigurer{
}
```

- `@EnableWebMvc` + `WebMvcConfigurer` — `@Bean`  可以**全面接管SpringMVC**，所有规则全部自己重新配置； 实现定制和扩展功能（**高级功能，初学者退避三舍**）。包括静态资源都会失效（所有的都需要我们自己去定义，慎用）

  ![image-20220315090712784](https://image.imxyu.cn/file/image-20220315090712784.png)

  

  - 原理：
    1. `WebMvcAutoConfiguration`默认的SpringMVC的自动配置功能类，如静态资源、欢迎页等。
    2. 一旦使用 `@EnableWebMvc` ，会`@Import(DelegatingWebMvcConfiguration.class)`。
    3. `DelegatingWebMvcConfiguration`的作用，只保证SpringMVC最基本的使用
       - 把所有系统中的`WebMvcConfigurer`拿过来，所有功能的定制都是这些`WebMvcConfigurer`合起来一起生效。
       - 自动配置了一些非常底层的组件，如`RequestMappingHandlerMapping`，这些组件依赖的组件都是从容器中获取如。
       - 而我们@EnableWebMvc引入的`public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport`。
    4. `WebMvcAutoConfiguration`里面的配置要能生效必须  `@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)`。
    5. @EnableWebMvc 导致了WebMvcAutoConfiguration  没有生效。

![image-20220315090443485](https://image.imxyu.cn/file/image-20220315090443485.png)





### 原理分析套路

场景starter - `xxxxAutoConfiguration` - 导入xxx组件 - 绑定`xxxProperties` - 绑定配置文件项。

> 之后的场景整合，我们只需要引入相应的启动器，然后看引入的自动配置类中给我们导入了什么组件，绑定了什么配置文件，我们只需要修改配置文件的前缀项就可以了



# 数据访问

## 数据源的自动配置

### 导入JDBC场景

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

接着导入数据库驱动包（MySQL为例）。

```xml
<!--默认版本：-->
<mysql.version>8.0.22</mysql.version>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <!--<version>5.1.49</version>-->
</dependency>

<!--
想要修改版本
1、直接依赖引入具体版本（maven的就近依赖原则）
2、重新声明版本（maven的属性的就近优先原则）
-->
<properties>
    <java.version>1.8</java.version>
    <mysql.version>5.1.49</mysql.version>
</properties>
```



### 相关数据源配置类



DataSource与连接池的关系是:

**DataSource内部封装了一个连接池**,当你获取DataSource的时候,它已经敲敲的与数据库建立了多个Connection,并将这些Connection放入了连接池

DataSource利用连接池缓存Connection,以达到系统效率的提升,资源的重复利用



引入的配置类：

- `DataSourceAutoConfiguration` ： 数据源的自动配置。

  - 修改数据源相关的配置：`spring.datasource`。
  - **数据源（连接池）的配置，是自己容器中没有DataSource才自动配置的**。
  - 底层配置好的数据源是：`HikariDataSource`。
- `DataSourceTransactionManagerAutoConfiguration`： 事务管理器的自动配置。
- `JdbcTemplateAutoConfiguration`： `JdbcTemplate`的自动配置，可以来对数据库进行CRUD。
  - 可以修改前缀为`spring.jdbc`的配置项来修改`JdbcTemplate`。
  - `@Bean @Primary JdbcTemplate`：Spring容器中有这个`JdbcTemplate`组件，使用`@Autowired`。
- `JndiDataSourceAutoConfiguration`： JNDI的自动配置。
- `XADataSourceAutoConfiguration`： 分布式事务相关的。

![image-20220315092415511](https://image.imxyu.cn/file/image-20220315092415511.png)

![image-20220315092355788](https://image.imxyu.cn/file/image-20220315092355788.png)

### 修改配置项

添加

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db_account
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
```



### 单元测试数据源

因为容器中自动给我配置了JdbcTemplate	，我们可以直接使用jdbcTemplate 操作数据库

![image-20220315092545351](https://image.imxyu.cn/file/image-20220315092545351.png)

同时springboot项目在创建的时候已经在test文件夹下创建好了相应的测试类，我们直接使用就行

![image-20220315092613317](https://image.imxyu.cn/file/image-20220315092613317.png)



```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.JdbcTemplate;

@SpringBootTest
class Boot05WebAdminApplicationTests {

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Test//用@org.junit.Test会报空指针异常，可能跟JUnit新版本有关
    void contextLoads() {
//        jdbcTemplate.queryForObject("select * from account_tbl")
//        jdbcTemplate.queryForList("select * from account_tbl",)
        Long aLong = jdbcTemplate.queryForObject("select count(*) from account_tbl", Long.class);
        log.info("记录总数：{}",aLong);
    }

}
```

## 使用druid数据源

上面当我们引入jdbc 场景之后默认使用的是hakri 数据源，如果我们想引入第三方数据源Druid是什么？

它是数据库连接池，它能够提供强大的监控和扩展功能。

[官方文档 - Druid连接池介绍](https://github.com/alibaba/druid/wiki/Druid%E8%BF%9E%E6%8E%A5%E6%B1%A0%E4%BB%8B%E7%BB%8D)



Spring Boot整合第三方技术的两种方式：

- 自定义

- 找starter场景

### 自定义方式



以前我们需要使用xml的时候需要这么配置，现在看到这个bean的xml 标签就相当于一个@Bean的方法，我们可以在配置类中创建

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
		destroy-method="close">
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
		<property name="maxActive" value="20" />
		<property name="initialSize" value="1" />
		<property name="maxWait" value="60000" />
		<property name="minIdle" value="1" />
		<property name="timeBetweenEvictionRunsMillis" value="60000" />
		<property name="minEvictableIdleTimeMillis" value="300000" />
		<property name="testWhileIdle" value="true" />
		<property name="testOnBorrow" value="false" />
		<property name="testOnReturn" value="false" />
		<property name="poolPreparedStatements" value="true" />
		<property name="maxOpenPreparedStatements" value="20" />

```

> 并且我们在创建的时候发现这个数据源要设置的信息，和我们的自动配置类中的数据源的字段是一样的
>
> 那么我们可以直接复用配置文件中的，自动绑定到对象上

**添加依赖**：

```java
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.17</version>
</dependency>
```



**配置Druid数据源**：



通过上面spring 自动配置类来看，这个hikari 数据源只有在当前容器中没有数据源的时候才会帮我们放里面，因此我们只需要配置我们自己的数据源即可

![image-20220315092355788](https://image.imxyu.cn/file/image-20220315092355788.png)

```java
@Configuration
public class MyConfig {

    @Bean
    @ConfigurationProperties("spring.datasource")//复用配置文件的数据源配置，会自动绑定到这个对象上，就不需要我们自己set了
    public DataSource dataSource() throws SQLException {
        DruidDataSource druidDataSource = new DruidDataSource();

//        druidDataSource.setUrl();  
//        druidDataSource.setUsername();
//        druidDataSource.setPassword();

        return druidDataSource;
    }
}
```

[更多配置项](https://github.com/alibaba/druid/wiki/DruidDataSource%E9%85%8D%E7%BD%AE)

**配置Druid的监控页功能**：

- Druid内置提供了一个`StatViewServlet`用于展示Druid的统计信息。[官方文档 - 配置_StatViewServlet配置](https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE)。这个`StatViewServlet`的用途包括：
  - 提供监控信息展示的html页面
  - 提供监控信息的JSON API

- Druid内置提供一个`StatFilter`，用于统计监控信息。[官方文档 - 配置_StatFilter](https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatFilter)
- `WebStatFilter`用于采集web-jdbc关联监控的数据，如SQL监控、URI监控。[官方文档 - 配置_配置WebStatFilter](https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_%E9%85%8D%E7%BD%AEWebStatFilter)
- Druid提供了`WallFilter`，它是基于SQL语义分析来实现防御SQL注入攻击的。[官方文档 - 配置 wallfilter](https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE-wallfilter)

```java
@Configuration
public class MyConfig {

    @Bean
    @ConfigurationProperties("spring.datasource")
    public DataSource dataSource() throws SQLException {
        DruidDataSource druidDataSource = new DruidDataSource();

        //加入监控和防火墙功能功能
        druidDataSource.setFilters("stat,wall");
        
        return druidDataSource;
    }
    
    /**
     * 配置 druid的监控页功能，这就是注册原生servlet 的方式，配置好了之后可以直接访问localhost:8080/druid 访问，同样我们可以添加用户名密码
     * @return
     */
    @Bean
    public ServletRegistrationBean statViewServlet(){
        StatViewServlet statViewServlet = new StatViewServlet();
        ServletRegistrationBean<StatViewServlet> registrationBean = 
            new ServletRegistrationBean<>(statViewServlet, "/druid/*");

        //监控页账号密码：
        registrationBean.addInitParameter("loginUsername","admin");
        registrationBean.addInitParameter("loginPassword","123456");

        return registrationBean;
    }
    
     /**
     * WebStatFilter 用于采集web-jdbc关联监控的数据。
     */
    @Bean
    public FilterRegistrationBean webStatFilter(){
        WebStatFilter webStatFilter = new WebStatFilter();

        FilterRegistrationBean<WebStatFilter> filterRegistrationBean = new FilterRegistrationBean<>(webStatFilter);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/*"));
        filterRegistrationBean.addInitParameter("exclusions","*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");

        return filterRegistrationBean;
    }
    
}
```

> 以上的所有信息都可以根据文档转换成相应的@Bean对象即可

## druid数据源starter整合方式

[官方文档 - Druid Spring Boot Starter]()

**引入依赖**：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.17</version>
</dependency>
```

可以看到这个自动配置类会在我们的数据源的自动配置类之前，因此当它创建完了druid数据源之后spring自己的hikri数据源就不会注入了

我们只需要通过这两个properties 进行绑定就好，会自动绑定到这些类上

![image-20220315100331440](https://image.imxyu.cn/file/image-20220315100331440.png)



**分析自动配置**：

- 扩展配置项 `spring.datasource.druid`
- 自动配置类`DruidDataSourceAutoConfigure`
- `DruidSpringAopConfiguration.class`,  监控SpringBean的；配置项：`spring.datasource.druid.aop-patterns`
- `DruidStatViewServletConfiguration.class`, 监控页的配置。`spring.datasource.druid.stat-view-servlet`默认开启。
- `DruidWebStatFilterConfiguration.class`，web监控配置。`spring.datasource.druid.web-stat-filter`默认开启。
- `DruidFilterConfiguration.class`所有Druid的filter的配置：

```java
private static final String FILTER_STAT_PREFIX = "spring.datasource.druid.filter.stat";
private static final String FILTER_CONFIG_PREFIX = "spring.datasource.druid.filter.config";
private static final String FILTER_ENCODING_PREFIX = "spring.datasource.druid.filter.encoding";
private static final String FILTER_SLF4J_PREFIX = "spring.datasource.druid.filter.slf4j";
private static final String FILTER_LOG4J_PREFIX = "spring.datasource.druid.filter.log4j";
private static final String FILTER_LOG4J2_PREFIX = "spring.datasource.druid.filter.log4j2";
private static final String FILTER_COMMONS_LOG_PREFIX = "spring.datasource.druid.filter.commons-log";
private static final String FILTER_WALL_PREFIX = "spring.datasource.druid.filter.wall";
```

**配置示例**：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db_account
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver

    druid:
      aop-patterns: com.atguigu.admin.*  #监控SpringBean
      filters: stat,wall     # 底层开启功能，stat（sql监控），wall（防火墙）

      stat-view-servlet:   # 配置监控页功能
        enabled: true
        login-username: admin
        login-password: admin
        resetEnable: false

      web-stat-filter:  # 监控web
        enabled: true
        urlPattern: /*
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'


      filter:
        stat:    # 对上面filters里面的stat的详细配置
          slow-sql-millis: 1000 # 慢sql，对1000ms以上的作为慢sql
          logSlowSql: true
          enabled: true
        wall:
          enabled: true
          config:
            drop-table-allow: false
```

### 整合MyBatis

#### 数据访问-整合MyBatis-配置版

[MyBatis的GitHub仓库](https://github.com/mybatis)

[MyBatis官方](https://mybatis.org/mybatis-3/zh/index.html)

**starter的命名方式**：

1. SpringBoot官方的Starter：`spring-boot-starter-*`
2. 第三方的： `*-spring-boot-starter`

**引入依赖**：

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
```

**配置模式**:

给我们自动配置了如下：

- 全局配置文件

- `SqlSessionFactory`：自动配置好了

- `SqlSession`：自动配置了`SqlSessionTemplate` 组合了`SqlSession`（底层操作数据库用的就是sqlSession）

- `@Import(AutoConfiguredMapperScannerRegistrar.class)`

- `Mapper`： 只要我们写的操作MyBatis的接口标准了`@Mapper`就会被自动扫描进来

```java
@EnableConfigurationProperties(MybatisProperties.class) ： MyBatis配置项绑定类。
@AutoConfigureAfter({ DataSourceAutoConfiguration.class, MybatisLanguageDriverAutoConfiguration.class })
public class MybatisAutoConfiguration{
    ...
}

@ConfigurationProperties(prefix = "mybatis")
public class MybatisProperties{
    ...
}
```

---

**配置文件**：

```yaml
spring:
  datasource:
    username: root
    password: 1234
    url: jdbc:mysql://localhost:3306/my
    driver-class-name: com.mysql.jdbc.Driver

# 配置mybatis规则
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml  #全局配置文件位置
  mapper-locations: classpath:mybatis/*.xml  #sql映射文件位置
```



**mybatis-config.xml**:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	
    <!-- 由于Spring Boot自动配置缘故，此处不必配置，只用来做做样，这个配置文件不写也可以。-->
</configuration>
```



**Mapper接口**：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lun.boot.mapper.UserMapper">

    <select id="getUser" resultType="com.lun.boot.bean.User">
        select * from user where id=#{id}
    </select>
</mapper>
```

```java
import com.lun.boot.bean.User;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface UserMapper {
    public User getUser(Integer id);
}
```



**POJO**：

```java
public class User {
    private Integer id;
    private String name;
    
	//getters and setters...
}
```



**DB**：

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4;
```



**Controller and Service**：

```java
@Controller
public class UserController {

    @Autowired
    private UserService userService;

    @ResponseBody
    @GetMapping("/user/{id}")
    public User getUser(@PathVariable("id") Integer id){

        return userService.getUser(id);
    }

}
```

```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;//IDEA下标红线，可忽视这红线

    public User getUser(Integer id){
        return userMapper.getUser(id);
    }

}
```



配置`private Configuration configuration;` 也就是配置`mybatis.configuration`相关的，就是相当于改mybatis全局配置文件中的值。（也就是说配置了`mybatis.configuration`，就不需配置mybatis全局配置文件了）

```yaml
# 配置mybatis规则
mybatis:
  mapper-locations: classpath:mybatis/mapper/*.xml
  # 可以不写全局配置文件，所有全局配置文件的配置都放在configuration配置项中了。
  # config-location: classpath:mybatis/mybatis-config.xml
  configuration:
    map-underscore-to-camel-case: true
```



**小结**

- 导入MyBatis官方Starter。
- 编写Mapper接口，需`@Mapper`注解。
- 编写SQL映射文件并绑定Mapper接口。
- 在`application.yaml`中指定Mapper配置文件的所处位置，以及指定全局配置文件的信息 （建议：**配置在`mybatis.configuration`**）。

#### 数据访问-整合MyBatis-注解配置混合版



**注解与配置混合搭配，干活不累**：

```java
@Mapper
public interface UserMapper {
    public User getUser(Integer id);

    @Select("select * from user where id=#{id}")
    public User getUser2(Integer id);

    public void saveUser(User user);

    @Insert("insert into user(`name`) values(#{name})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    public void saveUser2(User user);

}

```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lun.boot.mapper.UserMapper">

    <select id="getUser" resultType="com.lun.boot.bean.User">
        select * from user where id=#{id}
    </select>

    <insert id="saveUser" useGeneratedKeys="true" keyProperty="id">
        insert into user(`name`) values(#{name})
    </insert>

</mapper>
```



- 简单DAO方法就写在注解上。复杂的就写在配置文件里。

- 使用`@MapperScan("com.lun.boot.mapper")` 简化，Mapper接口就可以不用标注`@Mapper`注解。

  

```java
@MapperScan("com.lun.boot.mapper")
@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }

}
```



**最佳实战：**

- 引入mybatis-starter
- **配置application.yaml中，指定mapper-location位置即可**
- 编写Mapper接口并标注@Mapper注解
- 简单方法直接注解方式
- 复杂方法编写mapper.xml进行绑定映射
- *@MapperScan("com.atguigu.admin.mapper") 简化，其他的接口就可以不用标注@Mapper注解*

## 整合MyBatisPlus操作数据库

[IDEA的MyBatis的插件 - MyBatisX](https://plugins.jetbrains.com/plugin/10119-mybatisx)

[MyBatisPlus官网](https://baomidou.com/)

[MyBatisPlus官方文档](https://baomidou.com/guide/)

### MyBatisPlus是什么

[MyBatis-Plus](https://github.com/baomidou/mybatis-plus)（简称 MP）是一个 [MyBatis](http://www.mybatis.org/mybatis-3/)的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

---

添加依赖：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.1</version>
</dependency>
```

- `MybatisPlusAutoConfiguration`配置类，`MybatisPlusProperties`配置项绑定。

- `SqlSessionFactory`自动配置好，底层是容器中默认的数据源。

- `mapperLocations`自动配置好的，有默认值`classpath*:/mapper/**/*.xml`，这表示任意包的类路径下的所有mapper文件夹下任意路径下的所有xml都是sql映射文件。  建议以后sql映射文件放在 mapper下。

- 容器中也自动配置好了`SqlSessionTemplate`。

- `@Mapper` 标注的接口也会被自动扫描，建议直接 `@MapperScan("com.lun.boot.mapper")`批量扫描。
- MyBatisPlus**优点**之一：只需要我们的Mapper继承MyBatisPlus的`BaseMapper` 就可以拥有CRUD能力，减轻开发工作。

![image-20220315111237461](https://image.imxyu.cn/file/image-20220315111237461.png)

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.lun.hellomybatisplus.model.User;

public interface UserMapper extends BaseMapper<User> {

}
```



### CRUD实验-数据列表展示

[官方文档 - CRUD接口](https://baomidou.com/guide/crud-interface.html)

使用MyBatis Plus提供的`IService`，`ServiceImpl`，减轻Service层开发工作。

```java
import com.lun.hellomybatisplus.model.User;
import com.baomidou.mybatisplus.extension.service.IService;

import java.util.List;

/**
 *  Service 的CRUD也不用写了
 */
public interface UserService extends IService<User> {
}
```

```java
import com.lun.hellomybatisplus.model.User;
import com.lun.hellomybatisplus.mapper.UserMapper;
import com.lun.hellomybatisplus.service.UserService;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserServiceImpl extends ServiceImpl<UserMapper,User> implements UserService {
}
```

### CRUD实验-分页、删除

添加**分页插件**：

```java
@Configuration
public class MyBatisConfig {


    /**
     * MybatisPlusInterceptor
     * @return
     */
    @Bean
    public MybatisPlusInterceptor paginationInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
        // paginationInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        // paginationInterceptor.setLimit(500);
        // 开启 count 的 join 优化,只针对部分 left join

        //这是分页拦截器
        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
        paginationInnerInterceptor.setOverflow(true);
        paginationInnerInterceptor.setMaxLimit(500L);
        mybatisPlusInterceptor.addInnerInterceptor(paginationInnerInterceptor);

        return mybatisPlusInterceptor;
    }
}
```



```html
<table class="display table table-bordered table-striped" id="dynamic-table">
    <thead>
        <tr>
            <th>#</th>
            <th>name</th>
            <th>age</th>
            <th>email</th>
            <th>操作</th>
        </tr>
    </thead>
    <tbody>
        <tr class="gradeX" th:each="user: ${users.records}">
            <td th:text="${user.id}"></td>
            <td>[[${user.name}]]</td>
            <td th:text="${user.age}">Win 95+</td>
            <td th:text="${user.email}">4</td>
            <td>
                <a th:href="@{/user/delete/{id}(id=${user.id},pn=${users.current})}" 
                   class="btn btn-danger btn-sm" type="button">删除</a>
            </td>
        </tr>
    </tfoot>
</table>

<div class="row-fluid">
    <div class="span6">
        <div class="dataTables_info" id="dynamic-table_info">
            当前第[[${users.current}]]页  总计 [[${users.pages}]]页  共[[${users.total}]]条记录
        </div>
    </div>
    <div class="span6">
        <div class="dataTables_paginate paging_bootstrap pagination">
            <ul>
                <li class="prev disabled"><a href="#">← 前一页</a></li>
                <li th:class="${num == users.current?'active':''}" 
                    th:each="num:${#numbers.sequence(1,users.pages)}" >
                    <a th:href="@{/dynamic_table(pn=${num})}">[[${num}]]</a>
                </li>
                <li class="next disabled"><a href="#">下一页 → </a></li>
            </ul>
        </div>
    </div>
</div>

```

`#numbers`表示methods for formatting numeric objects.[link](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#expression-utility-objects)

**controller**

```java
@GetMapping("/user/delete/{id}")
public String deleteUser(@PathVariable("id") Long id,
                         @RequestParam(value = "pn",defaultValue = "1")Integer pn,
                         RedirectAttributes ra){

    userService.removeById(id);

    ra.addAttribute("pn",pn);
    return "redirect:/dynamic_table";
}

@GetMapping("/dynamic_table")
public String dynamic_table(@RequestParam(value="pn",defaultValue = "1") Integer pn,Model model){
    //表格内容的遍历

    //从数据库中查出user表中的用户进行展示

    //构造分页参数
    Page<User> page = new Page<>(pn, 2);
    //调用page进行分页
    Page<User> userPage = userService.page(page, null);

    model.addAttribute("users",userPage);

    return "table/dynamic_table";
}
```



## 整合Redis环境

### 准备阿里云Redis环境

**添加依赖**：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!--导入jedis-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

- `RedisAutoConfiguration`自动配置类，RedisProperties 属性类 --> spring.redis.xxx是对redis的配置。
- 连接工厂`LettuceConnectionConfiguration`、`JedisConnectionConfiguration`是准备好的。
- 自动注入了`RedisTemplate<Object, Object>`，`xxxTemplate`。
- 自动注入了`StringRedisTemplate`，key，value都是String
- 底层只要我们使用`StringRedisTemplate`、`RedisTemplate`就可以操作Redis。



**外网Redis环境搭建**：

1. 阿里云按量付费Redis，其中选择**经典网络**。

2. 申请Redis的公网连接地址。

3. 修改白名单，允许`0.0.0.0/0`访问。



### Redis操作与统计小实验

相关Redis配置：

```java
spring:
  redis:
#   url: redis://lfy:Lfy123456@r-bp1nc7reqesxisgxpipd.redis.rds.aliyuncs.com:6379
    host: r-bp1nc7reqesxisgxpipd.redis.rds.aliyuncs.com
    port: 6379
    password: lfy:Lfy123456
    client-type: jedis
    jedis:
      pool:
        max-active: 10
#   lettuce:# 另一个用来连接redis的java框架
#      pool:
#        max-active: 10
#        min-idle: 5
```

测试Redis连接：

```java
@SpringBootTest
public class Boot05WebAdminApplicationTests {

    @Autowired
    StringRedisTemplate redisTemplate;


    @Autowired
    RedisConnectionFactory redisConnectionFactory;

    @Test
    void testRedis(){
        ValueOperations<String, String> operations = redisTemplate.opsForValue();

        operations.set("hello","world");

        String hello = operations.get("hello");
        System.out.println(hello);

        System.out.println(redisConnectionFactory.getClass());
    }

}
```



Redis Desktop Manager：可视化Redis管理软件。



URL统计拦截器：

```java
@Component
public class RedisUrlCountInterceptor implements HandlerInterceptor {

    @Autowired
    StringRedisTemplate redisTemplate;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String uri = request.getRequestURI();

        //默认每次访问当前uri就会计数+1
        redisTemplate.opsForValue().increment(uri);

        return true;
    }
}
```



注册URL统计拦截器：

```java
@Configuration
public class AdminWebConfig implements WebMvcConfigurer{

    @Autowired
    RedisUrlCountInterceptor redisUrlCountInterceptor;


    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        registry.addInterceptor(redisUrlCountInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**",
                        "/js/**","/aa/**");
    }
}
```



Filter、Interceptor 几乎拥有相同的功能？

 * Filter是Servlet定义的原生组件，它的好处是脱离Spring应用也能使用。
 * Interceptor是Spring定义的接口，可以使用Spring的自动装配等功能。



调用Redis内的统计数据：

```java
@Slf4j
@Controller
public class IndexController {

	@Autowired
    StringRedisTemplate redisTemplate;
    
	@GetMapping("/main.html")
    public String mainPage(HttpSession session,Model model){

        log.info("当前方法是：{}","mainPage");

        ValueOperations<String, String> opsForValue =
                redisTemplate.opsForValue();

        String s = opsForValue.get("/main.html");
        String s1 = opsForValue.get("/sql");

        model.addAttribute("mainCount",s);
        model.addAttribute("sqlCount",s1);

        return "main";
    }
}
```

# 单元测试

**Spring Boot 2.2.0 版本开始引入 JUnit 5 作为单元测试默认库**

[JUnit 5官方文档](https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations)

作为最新版本的JUnit框架，JUnit5与之前版本的JUnit框架有很大的不同。由三个不同子项目的几个不同模块组成。

**JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage**

- **JUnit Platform**: Junit Platform是在JVM上启动测试框架的基础，不仅支持Junit自制的测试引擎，其他测试引擎也都可以接入。

- **JUnit Jupiter**: JUnit Jupiter提供了JUnit5的新的编程模型，是JUnit5新特性的核心。内部包含了一个**测试引擎**，用于在Junit Platform上运行。

- **JUnit Vintage**: 由于JUint已经发展多年，为了照顾老的项目，JUnit Vintage提供了兼容JUnit4.x，JUnit3.x的测试引擎。

**注意**：

- **SpringBoot 2.4** 以上版本移除了默认对 Vintage 的依赖。如果需要兼容JUnit4需要自行引入（不能使用JUnit4的功能 @Test）

- JUnit 5’s Vintage已经从`spring-boot-starter-test`从移除。如果需要继续兼容Junit4需要自行引入Vintage依赖：

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

- 使用添加JUnit 5，添加对应的starter：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

- Spring的JUnit 5的基本单元测试模板（Spring的JUnit4的是`@SpringBootTest`+`@RunWith(SpringRunner.class)`）：

```java
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;//注意不是org.junit.Test（这是JUnit4版本的）
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class SpringBootApplicationTests {

    @Autowired
    private Component component;
    
    @Test
    //@Transactional 标注后连接数据库有回滚功能
    public void contextLoads() {
		Assertions.assertEquals(5, component.getFive());
    }
}
```

> 在spring 的整合单元测试中当我们标注了@SpringBootTest，我们不仅可以使用@Autowired 注解注入容器中的组件，还可以使用@Transactional
>
> 注意我们使用junit5 @Test方法要导入 org.junit.jupiter.api.Test中的，而不是org.junit.Test 这是4的（2.4以后已经默认不支持4了，需要自己导入依赖兼容）

## 单元测试-常用测试注解

[官方文档 - Annotations](https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations)

- **@Test**：表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试
- **@ParameterizedTest**：表示方法是参数化测试。
- **@RepeatedTest**：表示方法可重复执行。
- **@DisplayName**：为测试类或者测试方法设置展示名称。
- **@BeforeEach**：表示在**每个**单元测试**之前**执行。
- **@AfterEach**：表示在**每个**单元测试**之后**执行。
- **@BeforeAll**：表示在**所有**单元测试**之前**执行。
- **@AfterAll**：表示在**所有**单元测试**之后**执行。
- **@Tag**：表示单元测试类别，类似于JUnit4中的@Categories。
- **@Disabled**：表示测试类或测试方法不执行，类似于JUnit4中的@Ignore。
- **@Timeout**：表示测试方法运行如果超过了指定时间将会返回错误。
- **@ExtendWith**：为测试类或测试方法提供扩展类引用。

```java
import org.junit.jupiter.api.*;

@DisplayName("junit5功能测试类")
public class Junit5Test {


    @DisplayName("测试displayname注解")
    @Test
    void testDisplayName() {
        System.out.println(1);
        System.out.println(jdbcTemplate);
    }
    
    @ParameterizedTest
    @ValueSource(strings = { "racecar", "radar", "able was I ere I saw elba" })
    void palindromes(String candidate) {
        assertTrue(StringUtils.isPalindrome(candidate));
    }
    

    @Disabled
    @DisplayName("测试方法2")
    @Test
    void test2() {
        System.out.println(2);
    }

    @RepeatedTest(5)
    @Test
    void test3() {
        System.out.println(5);
    }

    /**
     * 规定方法超时时间。超出时间测试出异常
     *
     * @throws InterruptedException
     */
    @Timeout(value = 500, unit = TimeUnit.MILLISECONDS)
    @Test
    void testTimeout() throws InterruptedException {
        Thread.sleep(600);
    }


    @BeforeEach
    void testBeforeEach() {
        System.out.println("测试就要开始了...");
    }

    @AfterEach
    void testAfterEach() {
        System.out.println("测试结束了...");
    }

    @BeforeAll
    static void testBeforeAll() {
        System.out.println("所有测试就要开始了...");
    }

    @AfterAll
    static void testAfterAll() {
        System.out.println("所有测试以及结束了...");

    }

}
```



## 单元测试-断言机制

断言Assertion是测试方法中的核心部分，用来对测试需要满足的条件进行验证。这些断言方法都是org.junit.jupiter.api.Assertions的静态方法。检查业务逻辑返回的数据是否合理。所有的测试运行结束以后，会有一个详细的测试报告。

以后就不要写system.out 了，直接使用断言判断。

JUnit 5 内置的断言可以分成如下几个类别：

> 使用断言的一个优点就是当项目开发完成之后，可以通过maven 的test功能对所有用例都跑一遍，然后会给我们出一个单元测试报告
>
> 建议一个大的功能写完之后都要写一个单元测试



### 简单断言

用来对单个值进行简单的验证。如：

| 方法            | 说明                                 |
| --------------- | ------------------------------------ |
| assertEquals    | 判断两个对象或两个原始类型是否相等   |
| assertNotEquals | 判断两个对象或两个原始类型是否不相等 |
| assertSame      | 判断两个对象引用是否指向同一个对象   |
| assertNotSame   | 判断两个对象引用是否指向不同的对象   |
| assertTrue      | 判断给定的布尔值是否为 true          |
| assertFalse     | 判断给定的布尔值是否为 false         |
| assertNull      | 判断给定的对象引用是否为 null        |
| assertNotNull   | 判断给定的对象引用是否不为 null      |

```java
@Test
@DisplayName("simple assertion")
public void simple() {
     assertEquals(3, 1 + 2, "simple math");
     assertNotEquals(3, 1 + 1);

     assertNotSame(new Object(), new Object());
     Object obj = new Object();
     assertSame(obj, obj);

     assertFalse(1 > 2);
     assertTrue(1 < 2);

     assertNull(null);
     assertNotNull(new Object());
}
```



### 数组断言

通过 assertArrayEquals 方法来判断两个对象或原始类型的数组是否相等。

```java
@Test
@DisplayName("array assertion")
public void array() {
	assertArrayEquals(new int[]{1, 2}, new int[] {1, 2});
}
```



### 组合断言

`assertAll()`方法接受多个 `org.junit.jupiter.api.Executable` 函数式接口的实例作为要验证的断言，可以通过 lambda 表达式很容易的提供这些断言。

```java
@Test
@DisplayName("assert all")
public void all() {
 assertAll("Math",
    () -> assertEquals(2, 1 + 1),
    () -> assertTrue(1 > 0)
 );
}
```

### 异常断言

在JUnit4时期，想要测试方法的异常情况时，需要用`@Rule`注解的`ExpectedException`变量还是比较麻烦的。而JUnit5提供了一种新的断言方式`Assertions.assertThrows()`，配合函数式编程就可以进行使用。

```java
@Test
@DisplayName("异常测试")
public void exceptionTest() {
    ArithmeticException exception = Assertions.assertThrows(
           //扔出断言异常
            ArithmeticException.class, () -> System.out.println(1 % 0));
}
```

### 超时断言

JUnit5还提供了Assertions.assertTimeout()为测试方法设置了超时时间。

```java
@Test
@DisplayName("超时测试")
public void timeoutTest() {
    //如果测试方法时间超过1s将会异常
    Assertions.assertTimeout(Duration.ofMillis(1000), () -> Thread.sleep(500));
}
```

### 快速失败

通过 fail 方法直接使得测试失败。

```java
@Test
@DisplayName("fail")
public void shouldFail() {
	fail("This should fail");
}
```

最后可以使用test 测试所有的测试功能，如果我们使用了断言，就可以得到我们是否通过。

![image-20220315114440164](https://image.imxyu.cn/file/image-20220315114440164.png)

![image-20220315114632307](https://image.imxyu.cn/file/image-20220315114632307.png)

## 单元测试-前置条件

Unit 5 中的前置条件（assumptions【假设】）类似于断言，不同之处在于不满足的**断言assertions**会使得测试方法失败，而**不满足的前置条件只会使得测试方法的执行终止**。

前置条件可以看成是测试方法执行的前提，当该前提不满足时，就没有继续执行的必要。

```java
@DisplayName("前置条件")
public class AssumptionsTest {
    private final String environment = "DEV";

    @Test
    @DisplayName("simple")
    public void simpleAssume() {
        assumeTrue(Objects.equals(this.environment, "DEV"));
        assumeFalse(() -> Objects.equals(this.environment, "PROD"));
    }

    @Test
    @DisplayName("assume then do")
    public void assumeThenDo() {
        assumingThat(
            Objects.equals(this.environment, "DEV"),
            () -> System.out.println("In DEV")
        );
    }
}
```

`assumeTrue` 和 `assumFalse` 确保给定的条件为 `true` 或 `false`，不满足条件会使得测试执行终止。

`assumingThat` 的参数是表示条件的布尔值和对应的 Executable 接口的实现对象。只有条件满足时，`Executable` 对象才会被执行；当条件不满足时，测试执行并不会终止。



## 单元测试-嵌套测试

[官方文档 - Nested Tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests-nested)

JUnit 5 可以通过 Java 中的内部类和`@Nested` 注解实现嵌套测试，从而可以更好的把相关的测试方法组织在一起。在内部类中可以使用`@BeforeEach` 和`@AfterEach`注解，而且嵌套的层次没有限制。

```java
@DisplayName("A stack")
class TestingAStackDemo {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```



## 单元测试-参数化测试

[官方文档 - Parameterized Tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests)

参数化测试是JUnit5很重要的一个新特性，它使得用不同的参数多次运行测试成为了可能，也为我们的单元测试带来许多便利。

利用@ValueSource等注解，指定入参，我们将可以使用不同的参数进行多次单元测试，而不需要每新增一个参数就新增一个单元测试，省去了很多冗余代码。

利用**@ValueSource**等注解，指定入参，我们将可以使用不同的参数进行多次单元测试，而不需要每新增一个参数就新增一个单元测试，省去了很多冗余代码。

- **@ValueSource**: 为参数化测试指定入参来源，支持八大基础类以及String类型,Class类型
- **@NullSource**: 表示为参数化测试提供一个null的入参
- **@EnumSource**: 表示为参数化测试提供一个枚举入参
- **@CsvFileSource**：表示读取指定CSV文件内容作为参数化测试入参
- **@MethodSource**：表示读取指定方法的返回值作为参数化测试入参(注意方法返回需要是一个流)

当然如果参数化测试仅仅只能做到指定普通的入参还达不到让我觉得惊艳的地步。让我真正感到他的强大之处的地方在于他可以支持外部的各类入参。如:CSV,YML,JSON 文件甚至方法的返回值也可以作为入参。只需要去实现**`ArgumentsProvider`**接口，任何外部文件都可以作为它的入参。

```java
@ParameterizedTest
@ValueSource(strings = {"one", "two", "three"})
@DisplayName("参数化测试1")
public void parameterizedTest1(String string) {
    System.out.println(string);
    Assertions.assertTrue(StringUtils.isNotBlank(string));
}


@ParameterizedTest
@MethodSource("method")    //指定方法名
@DisplayName("方法来源参数")
public void testWithExplicitLocalMethodSource(String name) {
    System.out.println(name);
    Assertions.assertNotNull(name);
}

static Stream<String> method() {
    return Stream.of("apple", "banana");
}
```

### 迁移指南

[官方文档 - Migrating from JUnit 4](https://junit.org/junit5/docs/current/user-guide/#migrating-from-junit4)

在进行迁移的时候需要注意如下的变化：

- 注解在 `org.junit.jupiter.api` 包中，断言在 `org.junit.jupiter.api.Assertions` 类中，前置条件在 `org.junit.jupiter.api.Assumptions` 类中。
- 把`@Before` 和`@After` 替换成`@BeforeEach` 和`@AfterEach`。
- 把`@BeforeClass` 和`@AfterClass` 替换成`@BeforeAll` 和@AfterAll。
- 把`@Ignore` 替换成`@Disabled`。
- 把`@Category` 替换成`@Tag`。
- 把`@RunWith`、`@Rule` 和`@ClassRule` 替换成`@ExtendWith`。



# 指标监控

未来每一个微服务在云上部署以后，我们都需要对其进行监控、追踪、审计、控制等。SpringBoot就抽取了Actuator场景，使得我们每个微服务快速引用即可获得生产级别的应用监控、审计等功能。

[官方文档 - Spring Boot Actuator: Production-ready Features](https://docs.spring.io/spring-boot/docs/2.4.2/reference/htmlsingle/#production-ready)

**1.x与2.x的不同**：

- SpringBoot Actuator 1.x
  - 支持SpringMVC
  - 基于继承方式进行扩展
  - 层级Metrics配置
  - 自定义Metrics收集
  - 默认较少的安全策略

- SpringBoot Actuator 2.x
  - 支持SpringMVC、JAX-RS以及Webflux
  - 注解驱动进行扩展
  - 层级&名称空间Metrics
  - 底层使用MicroMeter，强大、便捷默认丰富的安全策略

### 如何使用

- 添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

- 访问`http://localhost:8080/actuator/**`。
- 暴露所有监控信息为HTTP。

```yaml
management:
  endpoints:
    enabled-by-default: true #暴露所有端点信息
    web:
      exposure:
        include: '*'  #以web方式暴露
```

- 测试例子
  - http://localhost:8080/actuator/beans
  - http://localhost:8080/actuator/configprops
  - http://localhost:8080/actuator/metrics
  - http://localhost:8080/actuator/metrics/jvm.gc.pause
  - http://localhost:8080/actuator/metrics/endpointName/detailPath



> actuator
>
> 英 [ˈæktjʊeɪtə]   美 [ˈæktjuˌeɪtər]
>
> n. 致动（促动，激励，调节）器；传动（装置，机构）；拖动装置；马达；操作机构；执行机构（元件）；（电磁铁）螺线管；操纵装置（阀门）；调速控制器；往复运动油（气）缸；作动筒

> metric
>
> 英 [ˈmetrɪk]   美 [ˈmetrɪk]
>
> adj. 米制的;公制的;按公制制作的;用公制测量的
>
> n. 度量标准;[数学]度量;诗体;韵文;诗韵



## 指标监控-常使用的端点及开启与禁用

### 常使用的端点

| ID                 | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `auditevents`      | 暴露当前应用程序的审核事件信息。需要一个`AuditEventRepository组件`。 |
| `beans`            | 显示应用程序中所有Spring Bean的完整列表。                    |
| `caches`           | 暴露可用的缓存。                                             |
| `conditions`       | 显示自动配置的所有条件信息，包括匹配或不匹配的原因。         |
| `configprops`      | 显示所有`@ConfigurationProperties`。                         |
| `env`              | 暴露Spring的属性`ConfigurableEnvironment`                    |
| `flyway`           | 显示已应用的所有Flyway数据库迁移。 需要一个或多个`Flyway`组件。 |
| `health`           | 显示应用程序运行状况信息。                                   |
| `httptrace`        | 显示HTTP跟踪信息（默认情况下，最近100个HTTP请求-响应）。需要一个`HttpTraceRepository`组件。 |
| `info`             | 显示应用程序信息。                                           |
| `integrationgraph` | 显示Spring `integrationgraph` 。需要依赖`spring-integration-core`。 |
| `loggers`          | 显示和修改应用程序中日志的配置。                             |
| `liquibase`        | 显示已应用的所有Liquibase数据库迁移。需要一个或多个`Liquibase`组件。 |
| `metrics`          | 显示当前应用程序的“指标”信息。                               |
| `mappings`         | 显示所有`@RequestMapping`路径列表。                          |
| `scheduledtasks`   | 显示应用程序中的计划任务。                                   |
| `sessions`         | 允许从Spring Session支持的会话存储中检索和删除用户会话。需要使用Spring Session的基于Servlet的Web应用程序。 |
| `shutdown`         | 使应用程序正常关闭。默认禁用。                               |
| `startup`          | 显示由`ApplicationStartup`收集的启动步骤数据。需要使用`SpringApplication`进行配置`BufferingApplicationStartup`。 |
| `threaddump`       | 执行线程转储。                                               |

如果您的应用程序是Web应用程序（Spring MVC，Spring WebFlux或Jersey），则可以使用以下附加端点：

| ID           | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| `heapdump`   | 返回`hprof`堆转储文件。                                      |
| `jolokia`    | 通过HTTP暴露JMX bean（需要引入Jolokia，不适用于WebFlux）。需要引入依赖`jolokia-core`。 |
| `logfile`    | 返回日志文件的内容（如果已设置`logging.file.name`或`logging.file.path`属性）。支持使用HTTP`Range`标头来检索部分日志文件的内容。 |
| `prometheus` | 以Prometheus服务器可以抓取的格式公开指标。需要依赖`micrometer-registry-prometheus`。 |

其中最常用的Endpoint：

- **Health：监控状况**
- **Metrics：运行时指标**
- **Loggers：日志记录**

### Health Endpoint

健康检查端点，我们一般用于在云平台，平台会定时的检查应用的健康状况，我们就需要Health Endpoint可以为平台返回当前应用的一系列组件健康状况的集合。

重要的几点：

- health endpoint返回的结果，应该是一系列健康检查后的一个汇总报告。
- 很多的健康检查默认已经自动配置好了，比如：数据库、redis等。
- 可以很容易的添加自定义的健康检查机制。

### Metrics Endpoint

提供详细的、层级的、空间指标信息，这些信息可以被pull（主动推送）或者push（被动获取）方式得到：

- 通过Metrics对接多种监控系统。
- 简化核心Metrics开发。
- 添加自定义Metrics或者扩展已有Metrics。



### 开启与禁用Endpoints

- 默认所有的Endpoint除过shutdown都是开启的。
- 需要开启或者禁用某个Endpoint。配置模式为`management.endpoint.<endpointName>.enabled = true`

```yaml
management:
  endpoint:
    beans:
      enabled: true
```

- 或者禁用所有的Endpoint然后手动开启指定的Endpoint。

```yaml
management:
  endpoints:
    enabled-by-default: false
  endpoint:
    beans:
      enabled: true
    health:
      enabled: true
```



### 暴露Endpoints

支持的暴露方式

- HTTP：默认只暴露health和info。
- JMX：默认暴露所有Endpoint。
- 除过health和info，剩下的Endpoint都应该进行保护访问。如果引入Spring Security，则会默认配置安全访问规则。

| ID                 | JMX  | Web  |
| :----------------- | :--- | :--- |
| `auditevents`      | Yes  | No   |
| `beans`            | Yes  | No   |
| `caches`           | Yes  | No   |
| `conditions`       | Yes  | No   |
| `configprops`      | Yes  | No   |
| `env`              | Yes  | No   |
| `flyway`           | Yes  | No   |
| `health`           | Yes  | Yes  |
| `heapdump`         | N/A  | No   |
| `httptrace`        | Yes  | No   |
| `info`             | Yes  | Yes  |
| `integrationgraph` | Yes  | No   |
| `jolokia`          | N/A  | No   |
| `logfile`          | N/A  | No   |
| `loggers`          | Yes  | No   |
| `liquibase`        | Yes  | No   |
| `metrics`          | Yes  | No   |
| `mappings`         | Yes  | No   |
| `prometheus`       | N/A  | No   |
| `scheduledtasks`   | Yes  | No   |
| `sessions`         | Yes  | No   |
| `shutdown`         | Yes  | No   |
| `startup`          | Yes  | No   |
| `threaddump`       | Yes  | No   |

若要更改公开的Endpoint，请配置以下的包含和排除属性：

| Property                                    | Default        |
| :------------------------------------------ | :------------- |
| `management.endpoints.jmx.exposure.exclude` |                |
| `management.endpoints.jmx.exposure.include` | `*`            |
| `management.endpoints.web.exposure.exclude` |                |
| `management.endpoints.web.exposure.include` | `info, health` |

[官方文档 - Exposing Endpoints](https://docs.spring.io/spring-boot/docs/2.4.2/reference/htmlsingle/#production-ready-endpoints-exposing-endpoints)



## 指标监控-定制Endpoint

### 定制 Health 信息

```yaml
management:
    health:
      enabled: true
      show-details: always #总是显示详细信息。可显示每个模块的状态信息
```

通过实现`HealthIndicator `接口，或继承`MyComHealthIndicator `类。

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

}

/*
构建Health
Health build = Health.down()
                .withDetail("msg", "error service")
                .withDetail("code", "500")
                .withException(new RuntimeException())
                .build();
*/
```



```java
@Component
public class MyComHealthIndicator extends AbstractHealthIndicator {

    /**
     * 真实的检查方法
     * @param builder
     * @throws Exception
     */
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        //mongodb。  获取连接进行测试
        Map<String,Object> map = new HashMap<>();
        // 检查完成
        if(1 == 2){
//            builder.up(); //健康
            builder.status(Status.UP);
            map.put("count",1);
            map.put("ms",100);
        }else {
//            builder.down();
            builder.status(Status.OUT_OF_SERVICE);
            map.put("err","连接超时");
            map.put("ms",3000);
        }


        builder.withDetail("code",100)
                .withDetails(map);

    }
}
```



### 定制info信息

常用两种方式：

- 编写配置文件

```yaml
info:
  appName: boot-admin
  version: 2.0.1
  mavenProjectName: @project.artifactId@  #使用@@可以获取maven的pom文件值
  mavenProjectVersion: @project.version@
```

- 编写InfoContributor

```java
import java.util.Collections;

import org.springframework.boot.actuate.info.Info;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.stereotype.Component;

@Component
public class ExampleInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("example",
                Collections.singletonMap("key", "value"));
    }

}
```

http://localhost:8080/actuator/info 会输出以上方式返回的所有info信息



### 定制Metrics信息

[Spring Boot支持的metrics](https://docs.spring.io/spring-boot/docs/2.4.2/reference/htmlsingle/#production-ready-metrics-meter)

增加定制Metrics：

```java
class MyService{
    Counter counter;
    public MyService(MeterRegistry meterRegistry){
         counter = meterRegistry.counter("myservice.method.running.counter");
    }

    public void hello() {
        counter.increment();
    }
}
```

```java
//也可以使用下面的方式
@Bean
MeterBinder queueSize(Queue queue) {
    return (registry) -> Gauge.builder("queueSize", queue::size).register(registry);
}
```



### 定制Endpoint

```java
@Component
@Endpoint(id = "container")
public class DockerEndpoint {

    @ReadOperation
    public Map getDockerInfo(){
        return Collections.singletonMap("info","docker started...");
    }

    @WriteOperation
    private void restartDocker(){
        System.out.println("docker restarted....");
    }

}
```

场景：

- 开发ReadinessEndpoint来管理程序是否就绪。
- 开发LivenessEndpoint来管理程序是否存活。



## 指标监控-Boot Admin Server

[官方Github]()

[官方文档](https://codecentric.github.io/spring-boot-admin/2.3.1/#getting-started)

可视化指标监控

> What is Spring Boot Admin?
>
> codecentric’s Spring Boot Admin is a community project to manage and monitor your [Spring Boot](http://projects.spring.io/spring-boot/) ® applications. The applications register with our Spring Boot Admin Client (via HTTP) or are discovered using Spring Cloud ® (e.g. Eureka, Consul). The UI is just a Vue.js application on top of the Spring Boot Actuator endpoints.

[开始使用方法](https://codecentric.github.io/spring-boot-admin/2.3.1/#getting-started)



# 高级特性

## 多环境配置

为了方便多环境适配，Spring Boot简化了profile功能。

- 默认配置文件`application.yaml`任何时候都会加载。
- 指定环境配置文件`application-{env}.yaml`，`env`通常替代为`test`，
- 激活指定环境
  - 配置文件激活：`spring.profiles.active=prod`
  - 命令行激活：`java -jar xxx.jar --spring.profiles.active=prod  --person.name=haha`（修改配置文件的任意值，**命令行优先**）
- 默认配置与环境配置同时生效
- 同名配置项，profile配置优先

![image-20220315170031516](https://image.imxyu.cn/file/image-20220315170031516.png)

## @Profile条件装配功能



```java
@Data
@Component
@ConfigurationProperties("person")//在配置文件中配置
public class Person{
    private String name;
    private Integer age;
}
```



---

```java
public interface Person {

   String getName();
   Integer getAge();

}

@Profile("test")//只有生产环境为test的时候才会生效，加载application-test.yaml里的
@Component
@ConfigurationProperties("person")
@Data
public class Worker implements Person {

    private String name;
    private Integer age;
}

@Profile(value = {"prod","default"})//只有生产环境为prod的时候才会生效，加载application-prod.yaml里的
@Component
@ConfigurationProperties("person")
@Data
public class Boss implements Person {

    private String name;
    private Integer age;
}
```



application-test.yaml

```yaml
person:
  name: test-张三

server:
  port: 7000
```



application-prod.yaml

```yaml
person:
  name: prod-张三

server:
  port: 8000
```



`application.properties` 这个配置文件中和指定的application-prod.yaml 文件都会生效

```properties
# 激活prod配置文件
spring.profiles.active=prod
```



```java
@Autowired
private Person person;

@GetMapping("/")
public String hello(){
    //激活了prod，则返回Boss；激活了test，则返回Worker
    return person.getClass().toString();
}
```

---



@Profile还可以修饰在方法上：

```java
class Color {
}

@Configuration
public class MyConfig {

    @Profile("prod")
    @Bean
    public Color red(){
        return new Color();
    }

    @Profile("test")
    @Bean
    public Color green(){
        return new Color();
    }
}
```

> 在类上相当于类中所有的都会按照指定 的才会生效，在方法上只是当前这个方法

可以激活一组：

```properties
spring.profiles.active=production

spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq
```

## 外部化配置Config配置文件

[官方文档 - Externalized Configuration](https://docs.spring.io/spring-boot/docs/2.4.2/reference/htmlsingle/#boot-features-external-config)

Spring Boot uses a very particular `PropertySource` order that is designed to allow sensible overriding of values. Properties are considered in the following order (with values from lower items overriding earlier ones)（1优先级最低，14优先级最高）:

1. Default properties (specified by setting `SpringApplication.setDefaultProperties`).
2. [`@PropertySource`](https://docs.spring.io/spring/docs/5.3.3/javadoc-api/org/springframework/context/annotation/PropertySource.html) annotations on your `@Configuration` classes. Please note that such property sources are not added to the `Environment` until the application context is being refreshed. This is too late to configure certain properties such as `logging.*` and `spring.main.*` which are read before refresh begins.
3. Config data (such as `application.properties` files)
4. A `RandomValuePropertySource` that has properties only in `random.*`.
5. OS environment variables.
6. Java System properties (`System.getProperties()`).
7. JNDI attributes from `java:comp/env`.
8. `ServletContext` init parameters.
9. `ServletConfig` init parameters.
10. Properties from `SPRING_APPLICATION_JSON` (inline JSON embedded in an environment variable or system property).
11. Command line arguments.
12. `properties` attribute on your tests. Available on [`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/2.4.2/api/org/springframework/boot/test/context/SpringBootTest.html) and the [test annotations for testing a particular slice of your application](https://docs.spring.io/spring-boot/docs/2.4.2/reference/htmlsingle/#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests).
13. [`@TestPropertySource`](https://docs.spring.io/spring/docs/5.3.3/javadoc-api/org/springframework/test/context/TestPropertySource.html) annotations on your tests.
14. [Devtools global settings properties](https://docs.spring.io/spring-boot/docs/2.4.2/reference/htmlsingle/#using-boot-devtools-globalsettings) in the `$HOME/.config/spring-boot` directory when devtools is active.

```java
import org.springframework.stereotype.*;
import org.springframework.beans.factory.annotation.*;

@Component
public class MyBean {

    @Value("${name}")//以这种方式可以获得配置值
    private String name;

    // ...

}
```

---

### 配置文件加载顺序

- 外部配置源
  - Java属性文件。
  - YAML文件。
  - 环境变量。
  - 命令行参数。
- 配置文件查找位置
  1. classpath 根路径。
  2. classpath 根路径下config目录。
  3. jar包当前目录。
  4. jar包当前目录的config目录。
  5. /config子目录的直接子目录。
- 配置文件加载顺序：
  1. 当前jar包内部的`application.properties`和`application.yml`。
  2. 当前jar包内部的`application-{profile}.properties` 和 `application-{profile}.yml`。
  3. 引用的外部jar包的`application.properties`和`application.yml`。
  4. 引用的外部jar包的`application-{profile}.properties`和`application-{profile}.yml`。
- **指定环境优先，外部优先，后面的可以覆盖前面的同名配置项。**

## 高级特性-自定义starter细节



### starter启动原理

- starter的pom.xml引入autoconfigure依赖

```mermaid
graph LR
A[starter] -->B[autoconfigure]
B --> C[spring-boot-starter]

```

- autoconfigure包中配置使用`META-INF/spring.factories`中`EnableAutoConfiguration`的值，使得项目启动加载指定的自动配置类
- 编写自动配置类 `xxxAutoConfiguration` -> `xxxxProperties`

- - `@Configuration`
  - `@Conditional`
  - `@EnableConfigurationProperties`
  - `@Bean`
  - ......

- 引入starter --- `xxxAutoConfiguration` --- 容器中放入组件 ---- `绑定xxxProperties` ---- 配置项

### 自定义starter

- 目标：创建`HelloService`的自定义starter。

- 创建两个工程，分别命名为`hello-spring-boot-starter`（普通Maven工程），`hello-spring-boot-starter-autoconfigure`（需用用到Spring Initializr创建的Maven工程）。（这里直接创建空工程）

- `hello-spring-boot-starter`无需编写什么代码，只需让该工程引入`hello-spring-boot-starter-autoconfigure`依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lun</groupId>
    <artifactId>hello-spring-boot-starter</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.lun</groupId>
            <artifactId>hello-spring-boot-starter-autoconfigure</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

- `hello-spring-boot-starter-autoconfigure`的pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.4.2</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.lun</groupId>
	<artifactId>hello-spring-boot-starter-autoconfigure</artifactId>
	<version>1.0.0-SNAPSHOT</version>
	<name>hello-spring-boot-starter-autoconfigure</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
	</dependencies>
</project>
```

- 创建4个文件：
  - `com/lun/hello/auto/HelloServiceAutoConfiguration`
  - `com/lun/hello/bean/HelloProperties`
  - `com/lun/hello/service/HelloService`
  - `src/main/resources/META-INF/spring.factories`（这个文件会告诉spring 加载哪个自动配置类生效）

```java
import com.lun.hello.bean.HelloProperties;
import com.lun.hello.service.HelloService;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConditionalOnMissingBean(HelloService.class)
@EnableConfigurationProperties(HelloProperties.class)//默认HelloProperties放在容器中
public class HelloServiceAutoConfiguration {

    @Bean
    public HelloService helloService(){
        return new HelloService();
    }

}
```

```java
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("hello")
public class HelloProperties {
    private String prefix;
    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}

```

```java
import com.lun.hello.bean.HelloProperties;
import org.springframework.beans.factory.annotation.Autowired;


/**
 * 默认不要放在容器中
 */
public class HelloService {

    @Autowired
    private HelloProperties helloProperties;

    public String sayHello(String userName){
        return helloProperties.getPrefix() + ": " + userName + " > " + helloProperties.getSuffix();
    }
}
```

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.lun.hello.auto.HelloServiceAutoConfiguration
```

- 用maven插件，将两工程install到本地。

- 接下来，测试使用自定义starter，用Spring Initializr创建名为`hello-spring-boot-starter-test`工程，引入`hello-spring-boot-starter`依赖，其pom.xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.lun</groupId>
    <artifactId>hello-spring-boot-starter-test</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>hello-spring-boot-starter-test</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- 引入`hello-spring-boot-starter`依赖 -->
        <dependency>
            <groupId>com.lun</groupId>
            <artifactId>hello-spring-boot-starter</artifactId>
            <version>1.0.0-SNAPSHOT</version>
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

- 添加配置文件`application.properties`：

```properties
hello.prefix=hello
hello.suffix=666
```

- 添加单元测试类：

```java
import com.lun.hello.service.HelloService;//来自自定义starter
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class HelloSpringBootStarterTestApplicationTests {

    @Autowired
    private HelloService helloService;

    @Test
    void contextLoads() {
        // System.out.println(helloService.sayHello("lun"));
        Assertions.assertEquals("hello: lun > 666", helloService.sayHello("lun"));
    }

}
```

# 原理解析-SpringApplication创建初始化流程

## SpringBoot对象创建

Spring Boot应用的启动类：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloSpringBootStarterTestApplication {

    public static void main(String[] args) {
        SpringApplication.run(HelloSpringBootStarterTestApplication.class, args);
    }

}
```

```java
public class SpringApplication {
    
    ...
    
	public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
		return run(new Class<?>[] { primarySource }, args);
	}
    
    public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
		return new SpringApplication(primarySources).run(args);
	}
    
    //先看看new SpringApplication(primarySources)，下一节再看看run()
	public SpringApplication(Class<?>... primarySources) {
		this(null, primarySources);
	}
    
    public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
        //WebApplicationType是枚举类，有NONE,SERVLET,REACTIVE,下行webApplicationType是SERVLET
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
        
        //初始启动引导器，去spring.factories文件中找org.springframework.boot.Bootstrapper，但找不到实现Bootstrapper接口的类
		this.bootstrappers = new ArrayList<>(getSpringFactoriesInstances(Bootstrapper.class));
		
        //去spring.factories找 ApplicationContextInitializer
        setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
		
        //去spring.factories找 ApplicationListener(所有的getSpringFactoriesInstances() 这个方法都是从spring.factories文件中找东西)
        setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));

        this.mainApplicationClass = deduceMainApplicationClass();
	}
 	
    private Class<?> deduceMainApplicationClass() {
		try {
			StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
			for (StackTraceElement stackTraceElement : stackTrace) {
				if ("main".equals(stackTraceElement.getMethodName())) {
					return Class.forName(stackTraceElement.getClassName());
				}
			}
		}
		catch (ClassNotFoundException ex) {
			// Swallow and continue
		}
		return null;
	}
    
    ...
    
}
```

**spring.factories：**

```properties
...

# Application Context Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.rsocket.context.RSocketPortInfoApplicationContextInitializer,\
org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.ClearCachesApplicationListener,\
org.springframework.boot.builder.ParentContextCloserApplicationListener,\
org.springframework.boot.context.FileEncodingApplicationListener,\
org.springframework.boot.context.config.AnsiOutputApplicationListener,\
org.springframework.boot.context.config.DelegatingApplicationListener,\
org.springframework.boot.context.logging.LoggingApplicationListener,\
org.springframework.boot.env.EnvironmentPostProcessorApplicationListener,\
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener

...

```

> 可以看到上面都springboot 启动的时候找到一些**spring.factories：**定义好的东西放到了springboot 对象中

## SpringBoot对象创建后调用run方法

继续上一节，接着讨论`return new SpringApplication(primarySources).run(args)`的`run`方法

```java
public class SpringApplication {
    
    ...
    
	public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();//开始计时器
		stopWatch.start();//开始计时
        
        //1.
        //创建引导上下文（Context环境）createBootstrapContext()
        //获取到所有之前的 bootstrappers 挨个执行 intitialize() 来完成对引导启动器上下文环境设置
		DefaultBootstrapContext bootstrapContext = createBootstrapContext();
		
        //2.到最后该方法会返回这context
        ConfigurableApplicationContext context = null;
		
        //3.让当前应用进入headless模式
        configureHeadlessProperty();
        
        //4.获取所有 RunListener（运行监听器）,为了方便所有Listener进行事件感知
		SpringApplicationRunListeners listeners = getRunListeners(args);
		
        //5. 遍历 SpringApplicationRunListener 调用 starting 方法；
		// 相当于通知所有感兴趣系统正在启动过程的人，项目正在 starting。
        listeners.starting(bootstrapContext, this.mainApplicationClass);
		try {
            //6.保存命令行参数 ApplicationArguments
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			
            //7.准备环境
            ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
			configureIgnoreBeanInfo(environment);
			
            /*打印标志
              .   ____          _            __ _ _
             /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
            ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
             \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
              '  |____| .__|_| |_|_| |_\__, | / / / /
             =========|_|==============|___/=/_/_/_/
             :: Spring Boot ::                (v2.4.2)
            */
            Banner printedBanner = printBanner(environment);
            
            // 创建IOC容器（createApplicationContext（））
			// 根据项目类型webApplicationType（NONE,SERVLET,REACTIVE）创建容器，
			// 当前会创建 AnnotationConfigServletWebServerApplicationContext
			context = createApplicationContext();
			context.setApplicationStartup(this.applicationStartup);
            
            //8.准备ApplicationContext IOC容器的基本信息
			prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
			//9.刷新IOC容器,创建容器中的所有组件,Spring框架的内容
            refreshContext(context);
			//该方法没内容，大概为将来填入
			afterRefresh(context, applicationArguments);
			stopWatch.stop();//停止计时
			if (this.logStartupInfo) {//this.logStartupInfo默认是true
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
            //10.
			listeners.started(context);
            
            //11.调用所有runners
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
            //13.
			handleRunFailure(context, ex, listeners);
			throw new IllegalStateException(ex);
		}

		try {
            //12.
			listeners.running(context);
		}
		catch (Throwable ex) {
            //13.
			handleRunFailure(context, ex, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
 
    //1. 
    private DefaultBootstrapContext createBootstrapContext() {
		DefaultBootstrapContext bootstrapContext = new DefaultBootstrapContext();
		this.bootstrappers.forEach((initializer) -> initializer.intitialize(bootstrapContext));
		return bootstrapContext;
	}
    
    //3.
   	private void configureHeadlessProperty() {
        //this.headless默认为true
		System.setProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS,
				System.getProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS, Boolean.toString(this.headless)));
	}
    
    private static final String SYSTEM_PROPERTY_JAVA_AWT_HEADLESS = "java.awt.headless";
    
    //4.
    private SpringApplicationRunListeners getRunListeners(String[] args) {
		Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
		//getSpringFactoriesInstances 去 spring.factories 找 SpringApplicationRunListener
        return new SpringApplicationRunListeners(logger,
				getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args),
				this.applicationStartup);
	}
    
    //7.准备环境
    private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
			DefaultBootstrapContext bootstrapContext, ApplicationArguments applicationArguments) {
		// Create and configure the environment
        //返回或者创建基础环境信息对象，如：StandardServletEnvironment, StandardReactiveWebEnvironment
		ConfigurableEnvironment environment = getOrCreateEnvironment();
        //配置环境信息对象,读取所有的配置源的配置属性值。
		configureEnvironment(environment, applicationArguments.getSourceArgs());
		//绑定环境信息
        ConfigurationPropertySources.attach(environment);
        //7.1 通知所有的监听器当前环境准备完成
		listeners.environmentPrepared(bootstrapContext, environment);
		DefaultPropertiesPropertySource.moveToEnd(environment);
		configureAdditionalProfiles(environment);
		bindToSpringApplication(environment);
		if (!this.isCustomEnvironment) {
			environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
					deduceEnvironmentClass());
		}
		ConfigurationPropertySources.attach(environment);
		return environment;
	}
    
    //8.
    private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context,
			ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments, Banner printedBanner) {
		//保存环境信息
        context.setEnvironment(environment);
        //IOC容器的后置处理流程
		postProcessApplicationContext(context);
        //应用初始化器
		applyInitializers(context);
        //8.1 遍历所有的 listener 调用 contextPrepared。
        //EventPublishRunListenr通知所有的监听器contextPrepared
		listeners.contextPrepared(context);
		bootstrapContext.close(context);
		if (this.logStartupInfo) {
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}
		// Add boot specific singleton beans
		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
		beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
		if (printedBanner != null) {
			beanFactory.registerSingleton("springBootBanner", printedBanner);
		}
		if (beanFactory instanceof DefaultListableBeanFactory) {
			((DefaultListableBeanFactory) beanFactory)
					.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		if (this.lazyInitialization) {
			context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
		}
		// Load the sources
		Set<Object> sources = getAllSources();
		Assert.notEmpty(sources, "Sources must not be empty");
		load(context, sources.toArray(new Object[0]));
        //8.2
		listeners.contextLoaded(context);
	}

    //11.调用所有runners
    private void callRunners(ApplicationContext context, ApplicationArguments args) {
		List<Object> runners = new ArrayList<>();
        
        //获取容器中的 ApplicationRunner
		runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
		//获取容器中的  CommandLineRunner
        runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
        //合并所有runner并且按照@Order进行排序
		AnnotationAwareOrderComparator.sort(runners);
        //遍历所有的runner。调用 run 方法
		for (Object runner : new LinkedHashSet<>(runners)) {
			if (runner instanceof ApplicationRunner) {
				callRunner((ApplicationRunner) runner, args);
			}
			if (runner instanceof CommandLineRunner) {
				callRunner((CommandLineRunner) runner, args);
			}
		}
	}
    
    //13.
    private void handleRunFailure(ConfigurableApplicationContext context, Throwable exception,
			SpringApplicationRunListeners listeners) {
		try {
			try {
				handleExitCode(context, exception);
				if (listeners != null) {
                    //14.
					listeners.failed(context, exception);
				}
			}
			finally {
				reportFailure(getExceptionReporters(context), exception);
				if (context != null) {
					context.close();
				}
			}
		}
		catch (Exception ex) {
			logger.warn("Unable to close ApplicationContext", ex);
		}
		ReflectionUtils.rethrowRuntimeException(exception);
	}
    
    ...
}
```



```java
//2. new SpringApplication(primarySources).run(args) 最后返回的接口类型
public interface ConfigurableApplicationContext extends ApplicationContext, Lifecycle, Closeable {
    String CONFIG_LOCATION_DELIMITERS = ",; \t\n";
    String CONVERSION_SERVICE_BEAN_NAME = "conversionService";
    String LOAD_TIME_WEAVER_BEAN_NAME = "loadTimeWeaver";
    String ENVIRONMENT_BEAN_NAME = "environment";
    String SYSTEM_PROPERTIES_BEAN_NAME = "systemProperties";
    String SYSTEM_ENVIRONMENT_BEAN_NAME = "systemEnvironment";
    String APPLICATION_STARTUP_BEAN_NAME = "applicationStartup";
    String SHUTDOWN_HOOK_THREAD_NAME = "SpringContextShutdownHook";

    void setId(String var1);

    void setParent(@Nullable ApplicationContext var1);

    void setEnvironment(ConfigurableEnvironment var1);

    ConfigurableEnvironment getEnvironment();

    void setApplicationStartup(ApplicationStartup var1);

    ApplicationStartup getApplicationStartup();

    void addBeanFactoryPostProcessor(BeanFactoryPostProcessor var1);

    void addApplicationListener(ApplicationListener<?> var1);

    void setClassLoader(ClassLoader var1);

    void addProtocolResolver(ProtocolResolver var1);

    void refresh() throws BeansException, IllegalStateException;

    void registerShutdownHook();

    void close();

    boolean isActive();

    ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException;
}
```

```properties
#4.
#spring.factories
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```



![image.png](https://image.imxyu.cn/file/1607004823889-8373cea4-6305-40c1-af3b-921b071a28a8.png)

```java
//通过这个类在各个时期去调用所有的listeners （观察者模式）
class SpringApplicationRunListeners {

	private final Log log;

	private final List<SpringApplicationRunListener> listeners;

	private final ApplicationStartup applicationStartup;

	SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners,
			ApplicationStartup applicationStartup) {
		this.log = log;
		this.listeners = new ArrayList<>(listeners);
		this.applicationStartup = applicationStartup;
	}

    //5.遍历 SpringApplicationRunListener 调用 starting 方法；
	//相当于通知所有感兴趣系统正在启动过程的人，项目正在 starting。
	void starting(ConfigurableBootstrapContext bootstrapContext, Class<?> mainApplicationClass) {
		doWithListeners("spring.boot.application.starting", (listener) -> listener.starting(bootstrapContext),
				(step) -> {
					if (mainApplicationClass != null) {
						step.tag("mainApplicationClass", mainApplicationClass.getName());
					}
				});
	}
    
    //7.1
    void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
		doWithListeners("spring.boot.application.environment-prepared",
				(listener) -> listener.environmentPrepared(bootstrapContext, environment));
	}
    
    //8.1
    void contextPrepared(ConfigurableApplicationContext context) {
		doWithListeners("spring.boot.application.context-prepared", (listener) -> listener.contextPrepared(context));
	}
    
    //8.2
    void contextLoaded(ConfigurableApplicationContext context) {
		doWithListeners("spring.boot.application.context-loaded", (listener) -> listener.contextLoaded(context));
	}
    
    //10.
    void started(ConfigurableApplicationContext context) {
		doWithListeners("spring.boot.application.started", (listener) -> listener.started(context));
	}
    
    //12.
    void running(ConfigurableApplicationContext context) {
		doWithListeners("spring.boot.application.running", (listener) -> listener.running(context));
	}
    
    //14.
    void failed(ConfigurableApplicationContext context, Throwable exception) {
		doWithListeners("spring.boot.application.failed",
				(listener) -> callFailedListener(listener, context, exception), (step) -> {
					step.tag("exception", exception.getClass().toString());
					step.tag("message", exception.getMessage());
				});
	}
    
    private void doWithListeners(String stepName, Consumer<SpringApplicationRunListener> listenerAction,
			Consumer<StartupStep> stepAction) {
		StartupStep step = this.applicationStartup.start(stepName);
		this.listeners.forEach(listenerAction);
		if (stepAction != null) {
			stepAction.accept(step);
		}
		step.end();
	}
    
    ...
    
}
```

## 总结

![image-20220315161814843](https://image.imxyu.cn/file/image-20220315161814843.png)

![image-20220315161834141](https://image.imxyu.cn/file/image-20220315161834141.png)

```java
@FunctionalInterface
public interface ApplicationRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming application arguments
	 * @throws Exception on error
	 */
	void run(ApplicationArguments args) throws Exception;

}
```



```java
@FunctionalInterface
public interface CommandLineRunner {

	/**
	 * Callback used to run the bean.
	 * @param args incoming main method arguments
	 * @throws Exception on error
	 */
	void run(String... args) throws Exception;

}
```



# 自定义事件监听组件

通过上面的springboot的流程我们可以看到有很多监听器会在启动的时候被调用

`MyApplicationContextInitializer.java`

```java
import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;

public class MyApplicationContextInitializer implements ApplicationContextInitializer {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        System.out.println("MyApplicationContextInitializer ....initialize.... ");
    }
}
```



`MyApplicationListener.java`

```java
import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationListener;

public class MyApplicationListener implements ApplicationListener {
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("MyApplicationListener.....onApplicationEvent...");
    }
}
```



`MyApplicationRunner.java`

```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;


@Order(1)
@Component//放入容器
public class MyApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("MyApplicationRunner...run...");
    }
}
```



`MyCommandLineRunner.java`

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
/**
 * 应用启动做一个一次性事情
 */
@Order(2)
@Component//放入容器
public class MyCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("MyCommandLineRunner....run....");
    }
}
```



`MySpringApplicationRunListener.java`

```java
import org.springframework.boot.ConfigurableBootstrapContext;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringApplicationRunListener;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.ConfigurableEnvironment;

public class MySpringApplicationRunListener implements SpringApplicationRunListener {

    private SpringApplication application;
    public MySpringApplicationRunListener(SpringApplication application, String[] args){
        this.application = application;
    }

    @Override
    public void starting(ConfigurableBootstrapContext bootstrapContext) {
        System.out.println("MySpringApplicationRunListener....starting....");

    }


    @Override
    public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
        System.out.println("MySpringApplicationRunListener....environmentPrepared....");
    }


    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener....contextPrepared....");

    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener....contextLoaded....");
    }

    @Override
    public void started(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener....started....");
    }

    @Override
    public void running(ConfigurableApplicationContext context) {
        System.out.println("MySpringApplicationRunListener....running....");
    }

    @Override
    public void failed(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("MySpringApplicationRunListener....failed....");
    }
}
```



这三个组件是会去spring.factories 中找的，因此我们要创建一个文件写进去，而另外两个ApplicationRunner、CommandLineRunner会在容器中找，我们只需要加到容器中就行。执行流程就按照我们上面总结的那样。



注册`MyApplicationContextInitializer`，`MyApplicationListener`，`MySpringApplicationRunListener`:

`resources / META-INF / spring.factories`:

```properties
org.springframework.context.ApplicationContextInitializer=\
  com.lun.boot.listener.MyApplicationContextInitializer

org.springframework.context.ApplicationListener=\
  com.lun.boot.listener.MyApplicationListener

org.springframework.boot.SpringApplicationRunListener=\
  com.lun.boot.listener.MySpringApplicationRunListener
```

