上一节我们看了根据我们的请求拿到了处理器链（封装了目标方法和所有的拦截器）

而HandlerAdapter 就是一个**超级反射工具**，通过反射直接我们的目标方法的

这个适配器没有我们想的那么简单直接一个反射调用就完事了，它要处理方法中的各种参数赋值等，得到处理器进行方法执行后返回到相应的页面

下面我们来看一下

当拿到这个合适的handler之后，会去找相应的适配器，看哪个适配器能处理，我们点进去看一下

![image-20220221211220759](https://image.imxyu.cn/file/image-20220221211220759.png)

会拿到所有的适配器，这个适配器也是在八大组件初始化的时候加入的。同时我们也可以自己定义适配器去处理

![image-20220221211411868](https://image.imxyu.cn/file/image-20220221211411868.png)

我们来看一下这几个适配器

每个适配器中都有两个重要的方法

1、第一个是supports 方法，会判断当前的适配器能否处理这个handler

2、第二个是handle（），处理逻辑

**adapter适配器是策略模式+适配器模式的体现**

**在各个adapter策略中寻找适合的策略，然后调用handle方法，将handle请求转化成view 视图返回**

## HandlerMapping和HandlerAdapter交互

当这个handle处理器满足Adapter的support方法时，就会使用这个adapter适配器的handle（）方法处理这个请求

### HttpRequestHandlerAdapter

第一个适配器是HttpRequestHandlerAdapter，这个适配器会判断

![image-20220221211650959](https://image.imxyu.cn/file/image-20220221211650959.png)

它的工作supports方法就是判断这个handler看是否实现了HttpRequestHandler接口。

我们如果想让它工作可以写一个组件@Controller，名字以/开始，并且 实现HttpRequestHandler接口即可

### SimpleControllerHandlerAdapter

第二个适配器是SimpleControllerHandlerAdapter

它的support逻辑是实现了controller接口的handler

![image-20220221212047777](https://image.imxyu.cn/file/image-20220221212047777.png)

### HandlerFunctionAdapter

HandlerFunctionAdapter的条件是实现了HandlerFunction接口的handler

![image-20220221212256003](https://image.imxyu.cn/file/image-20220221212256003.png)

### RequestMappingHandlerAdapter

这个adapter的要求是HandlerMethod类型的

因此我们的之前的hello请求对应的就是方法，因此会走这个RequestMappingHandlerAdapter进行处理

![image-20220221212438117](https://image.imxyu.cn/file/image-20220221212438117.png)

### 满足HttpRequestHandlerAdapter的handler请求举例

下面我们来构造两个能满足HttpRequestHandlerAdapter的和SimpleControllerHandlerAdapter的请求

使用我们之前介绍的BeanNameUrlHandlerMapping来注册handler

```java
//BeanNameUrlHandlerMapping 创建好对象以后也要初始化，启动拿到容器中所有组件，看谁的名字是以/开始的，就把这个组件注册为处理器
@Controller("/helloReq") //BeanNameUrlHandlerMapping 就会把他注册进去
public class HelloHttpRequestHandler implements HttpRequestHandler {
   //启用 HttpRequestHandlerAdapter

   //处理请求
   @Override
   public void handleRequest(HttpServletRequest request,
                       HttpServletResponse response) throws ServletException, IOException {
      response.getWriter().write("HelloHttpRequestHandler....");
   }
}
```



```java
@org.springframework.stereotype.Controller("/helloSimple")
public class HelloSimpleController implements Controller {
	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		return null;
	}
}

```

我们之前说过三大HandlerMapping 中BeanNameUrlHandlerMapping这个是根据beanName放入路径的

具体的逻辑和RequestHandlerMapping之前一样，创建完对象后调用BeanNameUrlHandlerMapping初始化方法来加载

这个会加载所所有的是以"/" 斜杠开头的beanName。把这个beanName作为路径，保存到BeanNameUrlHandlerMapping的registry中

![image-20220221212813990](https://image.imxyu.cn/file/image-20220221212813990.png)

> 因此我们上面创建的两个类都会保存到BeanNameUrlHandlerMapping 中
>
> 当我们访问/helloReq和/helloSimple都会去BeanNameUrlHandlerMapping 这个中找到

比如说现在我们访问/helloReq 请求，就会在BeanNameUrlHandlerMapping中找到相应的处理器HelloHttpRequestHandler，然后寻找合适的adapter

![image-20220221213510622](https://image.imxyu.cn/file/image-20220221213510622.png)

然后寻找能处理HelloHttpRequestHandler的 处理器适配器，然后发现这个HttpRequestHandlerAdapter支持处理这个，因为HelloHttpRequestHandler实现了HttpRequestHandler接口，所以返回HttpRequestHandlerAdapter

![image-20220221213558840](https://image.imxyu.cn/file/image-20220221213558840.png)

然后就直接调用adapter的handler（）方法

![image-20220221213841352](https://image.imxyu.cn/file/image-20220221213841352.png)

这个handle方法会直接将我们的handler转换成HttpRequestHandler接口，然后调用handleRequest方法

![image-20220221213925695](https://image.imxyu.cn/file/image-20220221213925695.png)

这就是我们实现的handleRequest方法进行处理

![image-20220221214009225](https://image.imxyu.cn/file/image-20220221214009225.png)

>  同样的对于SimpleControllerHandlerAdapter这个适配器
>
> 也是判断满足controller 这个接口的可以处理，然后转换成controller 接口调用我们实现的handleRequest方法



当然我们也可以创建自己的adpter，当满足我们定义的adpater的support要求时，走我们的adapter的handle方法

对于这两种HttpRequestHandlerAdapter和SimpleControllerHandlerAdapter，直接转换成接口调用实现的handleRequest方法就行了

重点是RequestMappingHandlerAdapter，这个是处理**方法**的handler



我们来看一下这个接口

![image-20220222202814572](https://image.imxyu.cn/file/image-20220222202814572.png)

```java
@Nullable //适配的过程，把request、response和handler连接起来进行处理，处理完成得到 ModelAndView（页面解析过程就靠它）
ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
```

## 交互图

![image-20220221215251507](https://image.imxyu.cn/file/image-20220221215251507.png)

## 处理RequestMappingHandlerAdapter的请求

### HandlerAdapter中参数解析和返回值处理器何时准备的？

我们继续看如果是要执行一个方法，handlerAdapter是如何处理的？我们看之前的/hello请求

首先找到能处理的handler,返回handler链，然后开始寻找能处理这个handler的adapter



![image-20220222204955917](https://image.imxyu.cn/file/image-20220222204955917.png)

因为handler是一个方法，可以看到这里找到了RequestMappingHandlerAdapter来处理

![image-20220222205055907](https://image.imxyu.cn/file/image-20220222205055907.png)

接下来先执行拦截器的preHandle方法，这里我们先跳过，后面再说

我们重点来看一下真正执行目标方法的

```java
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

这个就是adapter适配器的作用，把request, response, 和handler转换成modelAndview返回的过程

> 的不同的适配器adapter的实现逻辑是不一样的，我们之前看了 HttpRequestHandlerAdapter和SimpleControllerHandlerAdapter的都是利用接口回调到我们自己实现的接口方法中处理，我们来看一下RequestMappingAdpater是怎么处理的？

![image-20220222205141687](https://image.imxyu.cn/file/image-20220222205141687.png)



![image-20220222205754171](https://image.imxyu.cn/file/image-20220222205754171.png)

这里上来会有一个锁

```
这是会话锁，每一个用户和服务器交互无论发了多少请求都只有一个会话，限制用户的线程
如果想控制用户的请求次数一个一个进入，防止用户做一些其他的操作扰乱程序。就可以用这个，一般用在高并发事务中
```

我们主要看mav = invokeHandlerMethod(request, response, handlerMethod); 目标方法的执行

![image-20220222205843935](https://image.imxyu.cn/file/image-20220222205843935.png)

这个方法比较长，我们一点一点来看

![image-20220222210100174](https://image.imxyu.cn/file/image-20220222210100174.png)

```java
//首先这句是将我们的request 和response包装到一个对象中，因为以后要经常用到这两个参数，同时对他们的方法进行一些增强（因此是装饰器模式）
ServletWebRequest webRequest = new ServletWebRequest(request, response);
```

```java
//这是一个数据绑定器，我们之前知道在方法的参数上可以直接写一个Person这样的对象，它可以将我们前台传入的所有有关Person的属性都包装放入到这个对象当中
WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
```

```java
//这个是准备了一个工厂，因为我们最后要返回ModelAndView Model（要交给页面的数据） View（我们要去的 视图）。model对象就是通过这个工厂产生的
ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);
```

```java
//对handlerMethod进行增强，可以快速得到handlerMethod中的一些属性比如参数和返回值什么的，这也是一个装饰器模式
ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod); 
```

```java
//下面这两个是参数解析器和返回值解析器，专门用来处理请求的参数和返回值
if (this.argumentResolvers != null) { //参数解析器
   invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
}
if (this.returnValueHandlers != null) { //返回值解析器（无论原生的返回值是什么最后都要返回modelAndView）
   invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
}
```

![image-20220222211128996](https://image.imxyu.cn/file/image-20220222211128996.png)

![image-20220222211143843](https://image.imxyu.cn/file/image-20220222211143843.png)

可以看到这个参数解析器有27个，对于不同类型的参数会用不同的解析器进行处理

返回值解析器有15个

> 那么我们就有疑惑了，这个两个组件是什么时候放入进来的呢？

答案和我们之前的handlerMapping一样，都是在默认的组件创建的时候利用initializingBean的afterPropertiesSet()方法，设置的这些

![image-20220222211417702](https://image.imxyu.cn/file/image-20220222211417702.png)

可以看到这个方法中的逻辑如下

![image-20220222211841363](https://image.imxyu.cn/file/image-20220222211841363.png)

### 参数解析器的工作流程

接下来准备了一个ModelAndViewContainer， 这是以后流程共享ModelAndView数据的临时存储容器

```
ModelAndViewContainer mavContainer = new ModelAndViewContainer();
```

![image-20220228143751852](https://image.imxyu.cn/file/image-20220228143751852.png)

接下来就是开始真正执行目标方法

```java
//真正开始执行目标方法
invocableMethod.invokeAndHandle(webRequest, mavContainer);
```

在这个方法中第一步就是目标方法的反射执行，我们点进去看一下

```java
Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
```

![image-20220228143846546](https://image.imxyu.cn/file/image-20220228143846546.png)

在这个方法中首先会找到方法参数的值，然后调用反射执行

我们重点来看一下是怎么找到方法参数的值的？

```java
//获取方法的请求参数
Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
```

![image-20220228143947683](https://image.imxyu.cn/file/image-20220228143947683.png)

```java
@GetMapping("/hello2") // 所有的xxxMapping都是RequestMapping
public String   sayHello(String name, //可以从请求参数中得到
                   @RequestParam("user")String user, //可以从请求参数中得到
                   HttpSession session, HttpServletRequest request, //原生的session对象
                   @RequestHeader("User-Agent") String  ua,
                   Model model,
                   Integer i,
                   RedirectAttributes ra){ //@RequestParam Map<String,Object> params：所有请求参数全封装进来
```

我们来模拟一个方法，可以看到这里面有很多参数。我们看一下是如何找到这些参数的值的

可以看到首先拿到方法中的这8个参数，接下来准备了一个数组，挨个确定这8个参数的值，for循环中就是确定方法参数值的过程，最后确定好了之后给我们return回去，我们看一下是如何确定的？

![image-20220228144451038](https://image.imxyu.cn/file/image-20220228144451038.png)

```java
if (!this.resolvers.supportsParameter(parameter)) { //支持这种参数的解析器也会被放到缓存，下一次进来，就不用27个人挨个判断
```

调用resolvers.supportsParameter（）方法

![image-20220228144757391](https://image.imxyu.cn/file/image-20220228144757391.png)

可以看到在这个方法中for循环每一个参数解析器，（一共27个），然后判断看哪个参数解析器能支持这个参数

![image-20220228144751977](https://image.imxyu.cn/file/image-20220228144751977.png)

参数解析器的接口和我们之前说的handlerMapping 一样，都是由两个方法。首先判断是否支持，如果支持的话使用另一个方法进行执行的过程

> 这也是策略模式的体现



![image-20220228144845886](https://image.imxyu.cn/file/image-20220228144845886.png)

其实判断support的过程也非常简单，就是看参数上是否标明了相应的注解。这里举例了其中一个的判断

![image-20220228145029770](https://image.imxyu.cn/file/image-20220228145029770.png)

当找到了合适的参数解析器，就把这个参数和解析器放到缓存中

下次同样的请求参数过来，就直接到缓存中取拿了。 spring底层有大量的缓存来加快框架的运行速度

![image-20220228145108317](https://image.imxyu.cn/file/image-20220228145108317.png)

返回后，就调用 this.resolvers.resolveArgument ，来执行解析参数的过程

![image-20220228145249448](https://image.imxyu.cn/file/image-20220228145249448.png)

这个resolver 是组合了这27个参数解析器和缓存

![image-20220228145456445](https://image.imxyu.cn/file/image-20220228145456445.png)

在这个方法中首先通过parmater，在cache中获取，然后进行赋值

![image-20220228145707259](https://image.imxyu.cn/file/image-20220228145707259.png)

这里面的方法就是调用原生的httprequest 和httpsession 等，帮我们赋值的过程

![image-20220228145922304](https://image.imxyu.cn/file/image-20220228145922304.png)

具体的就不看了

![image-20220228150039205](https://image.imxyu.cn/file/image-20220228150039205.png)



其他的几个参数的过程是一样的，拿到了所有的参数对应的值（保存在了这个object的数组中），直接调用反射执行方法

![image-20220228150241763](https://image.imxyu.cn/file/image-20220228150241763.png)

反射执行

![image-20220228150329527](https://image.imxyu.cn/file/image-20220228150329527.png)



> ```
> //方法的签名，到底能写那些？
> //详细参照 https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments
> //https://www.bilibili.com/video/BV19K4y1L7MT?p=32
> ```

比如

```
HttpSession session, HttpServletRequest request, //原生的request,session对象
@RequestHeader("User-Agent") String  ua, //spring能帮我们获取header头中的信息帮我们赋值到这里面，不需要我们手动获得request再去获取相应的头、判断空等
@RequestParam Map<String,Object> params //这个能直接把所有请求方法的参数的信息都保存到这个map的键值对中
```



### 返回值处理器的工作流程

当参数解析器处理完了之后，返回了我们index.jsp

接下来我们看一下返回值处理器是怎么工作的？

```
this.returnValueHandlers.handleReturnValue(
      returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
```

![image-20220228151800554](https://image.imxyu.cn/file/image-20220228151800554.png)

首先要找到合适的返回值处理器

```java
HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType); 
```

![image-20220228151907178](https://image.imxyu.cn/file/image-20220228151907178.png)

可以看到，又是一个for循环。在15个中进行判断，看是否支持这个返回值类型

![image-20220228151954985](https://image.imxyu.cn/file/image-20220228151954985.png)

![image-20220228152031815](https://image.imxyu.cn/file/image-20220228152031815.png)

这里找到了一个能处理的，我们是字符串类型，我们看一下这个是怎么判断的

![image-20220228152154464](https://image.imxyu.cn/file/image-20220228152154464.png)

![image-20220228152127014](https://image.imxyu.cn/file/image-20220228152127014.png)

可以看到这个处理器能支持的就是void或者字符串类型的返回值

下面就是他的处理器逻辑，可以看到如果返回值是字符串类型的，我们直接拿到返回的字符串设置到viewName 中，当作我们要返回的视图页面名

![image-20220228152217409](https://image.imxyu.cn/file/image-20220228152217409.png)

在这里面会有一个判断，看是否是重定向的方式，判断方式很简单，我们在路径前面可以通过redirect: 来标明

重定向会设置一个重定向标志位

```java
if (isRedirectViewName(viewName)) { //是否是重定向的方式  redirect:
```

![image-20220228165650703](https://image.imxyu.cn/file/image-20220228165650703.png)

```java
//SpringMVC的目标方法能写哪些返回值
//https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types
return "index.jsp";  //@PostMapping("/submit")  表单失败了  前一步，把表单中的数据放到ra中，  return  "redirect:form.jsp" //表单还能取到数据
```

### @ResponseBody注解

>  我们知道这个返回值是支持很多类型的，如果我们标了@Responsebody注解，我们要求返回的是字符串，而不是对应的视图，那么就不是使用上面的返回值处理器

我们注意到有一个RequestResponseBodyMethodProcessor 在这个view 处理器的前面，也就是说先会判断是否满足这个



![image-20220228152916640](https://image.imxyu.cn/file/image-20220228152916640.png)

判断是否标有这个注解

![image-20220228153336144](https://image.imxyu.cn/file/image-20220228153336144.png)

这就是这个处理器的执行过程

![image-20220228153641201](https://image.imxyu.cn/file/image-20220228153641201.png)

将我们的对象转换成json字符串。利用 HttpMessageConverter，直接读取输入输出流即可

![image-20220228153814969](https://image.imxyu.cn/file/image-20220228153814969.png)

### 封装一个ModelAndView对象返回

这样我们返回的页面也处理完了，然后就调用getModelAndView ，封装数据和视图

```java
return getModelAndView(mavContainer, modelFactory, webRequest);
```

![image-20220228171436395](https://image.imxyu.cn/file/image-20220228171436395.png)

这个就没什么好说的了，将我们上一步处理好的view视图创建一个modelAndView对象加入到modelAndView中

如果我们设置了model数据，同样会将我们的model加入进去，然后返回。

```java
//这里会有一个判断，如果是重定向的请求，会加入到session中，而不是单独的一个request域，这样数据可以共享
HttpServletRequest request = webRequest.getNativeRequest(HttpServletRequest.class);
if (request != null) { //重定向数据的共享，RedirectView。先把数据移到request，再把request移到session
   RequestContextUtils.getOutputFlashMap(request).putAll(flashAttributes);
}
```

![image-20220228171617540](https://image.imxyu.cn/file/image-20220228171617540.png)

### 没有返回视图给一个默认的路径

adapter就处理完了，返回了一个ModelAndView， 然后下面会有一个applyDefaultViewName(processedRequest, mv);

这个方法会判断，如果我们没有返回的视图名，则会给我们一个默认的视图名

![image-20220228172412407](https://image.imxyu.cn/file/image-20220228172412407.png)

这个默认的路径就是我们访问的路径

> 也就是说如果我们访问/hello ，没有指定返回的路径（比如说方法是void类型） 那么就会给我们设置一个默认的路径，这个路径就是我们访问的路径

![image-20220228182343744](https://image.imxyu.cn/file/image-20220228182343744.png)

接下来就是处理modelAndView 进行视图渲染，这里会catch到上面过程出现的异常，并且把异常也传了进去，我们下一节再讲 

![image-20220228184246700](https://image.imxyu.cn/file/image-20220228184246700.png)