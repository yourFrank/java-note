

视频地址：https://www.bilibili.com/video/BV19K4y1L7MT?from=search&seid=5309862794027495419&spm_id_from=333.337.0.0

# Springboot2核心功能

![image-20220312162532811](https://image.imxyu.cn/file/image-20220312162532811.png)

# 配置文件

## 文件类型

### properties

同以前的properties用法

#### 修改项目访问路径

![image-20220315165405228](https://image.imxyu.cn/file/image-20220315165405228.png)

![image-20220315165415458](https://image.imxyu.cn/file/image-20220315165415458.png)



![image-20220315165423686](https://image.imxyu.cn/file/image-20220315165423686.png)

### yaml

当properties 和yml 文件同时存在时，使用的是.properties的配置文件

#### 简介

YAML 是 "YAML Ain't Markup Language"（YAML 不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）。 



非常适合用来做以数据为中心的配置文件



#### 基本语法

- key: value；kv之间有空格
- 大小写敏感
- 使用缩进表示层级关系
- 缩进不允许使用tab，只允许空格
- 缩进的空格数不重要，只要相同层级的元素左对齐即可
- '#'表示注释
- 字符串无需加引号，如果要加，''与""表示字符串内容 会被 转义/不转义

#### 数据类型

- 字面量：单个的、不可再分的值。date、boolean、string、number、null

```yaml
k: v
```

- 对象：键值对的集合。map、hash、set、object 

```yaml
行内写法：  k: {k1:v1,k2:v2,k3:v3}
#或
k: 
	k1: v1
  k2: v2
  k3: v3
```

- 数组：一组按次序排列的值。array、list、queue

```yaml
行内写法：  k: [v1,v2,v3]
#或者
k:
 - v1
 - v2
 - v3
```

#### 示例

```java
@Data
public class Person {
	
	private String userName;
	private Boolean boss;
	private Date birth;
	private Integer age;
	private Pet pet;
	private String[] interests;
	private List<String> animal;
	private Map<String, Object> score;
	private Set<Double> salarys;
	private Map<String, List<Pet>> allPets;
}

@Data
public class Pet {
	private String name;
	private Double weight;
}
```

```yaml
# yaml表示以上对象
person:
  userName: zhangsan
  boss: false
  birth: 2019/12/12 20:12:33
  age: 18
  pet: 
    name: tomcat
    weight: 23.4
  interests: [篮球,游泳]
  animal: 
    - jerry
    - mario
  score:
    english: 
      first: 30
      second: 40
      third: 50
    math: [131,140,148]
    chinese: {first: 128,second: 136}
  salarys: [3999,4999.98,5999.99]
  allPets:
    sick:
      - {name: tom}
      - {name: jerry,weight: 47}
    health: [{name: mario,weight: 47}]
```

## 配置提示

我们在编写yml的时候配置文件和我们自定义的类绑定的时候一般没有提示 ，这样就很容易写错。我们可以添加下面一个依赖

同时再maven 插件打包的时候把这个依赖包排除出去

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>


 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.springframework.boot</groupId>
                            <artifactId>spring-boot-configuration-processor</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
```



![image-20220312164108884](https://image.imxyu.cn/file/image-20220312164108884.png)

# SpringMVC自动配置概览

以下内容来自官方文档的介绍：

Spring Boot provides auto-configuration for Spring MVC that **works well with most applications.(大多场景我们都无需自定义配置)**

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

- - 内容协商视图解析器和BeanName视图解析器

- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content))).

- - 静态资源（包括webjars）

- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.

- - 自动注册 `Converter，GenericConverter，Formatter `

- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-message-converters)).

- - 支持 `HttpMessageConverters` （后来我们配合内容协商理解原理）

- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-message-codes)).

- - 自动注册 `MessageCodesResolver` （国际化用）

- Static `index.html` support.

- - 静态index.html 页支持

- Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-favicon)).

- - 自定义 `Favicon`  

- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-web-binding-initializer)).

- - 自动使用 `ConfigurableWebBindingInitializer` ，（DataBinder负责将请求数据绑定到JavaBean上）

If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.2.9.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.

**不用@EnableWebMvc注解。使用** `**@Configuration**` **+** `**WebMvcConfigurer**` **自定义规则**



If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.

**声明** `**WebMvcRegistrations**` **改变默认底层组件**



If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.

**使用** `**@EnableWebMvc+@Configuration+DelegatingWebMvcConfiguration 全面接管SpringMVC**`

# 简单功能分析

## 静态资源访问

### 静态资源目录

只要静态资源放在类路径下： called `/static` (or `/public` or `/resources` or `/META-INF/resources`

访问 ： 当前项目根路径/ + 静态资源名 



原理： 静态映射/**。

请求进来，先去找Controller看能不能处理（比如说的@RequestMapping 的路径是bug.jpg ， 那么就会先走这个controller）。不能处理的所有请求又都交给静态资源处理器。静态资源也找不到则响应404页面

也可以改变默认的静态资源路径，`/static`，`/public`,`/resources`, `/META-INF/resources`失效

```yaml
spring:
  mvc:
    static-path-pattern: /res/**

  resources:
    static-locations: [classpath:/haha/]
```

当前项目 + static-path-pattern + 静态资源名 = 静态资源文件夹下找

### 静态资源访问前缀

默认无前缀

```yaml
spring:
  mvc:
    static-path-pattern: /res/**
```

> 这个还是很有用的，一般使用过滤器会拦截/**，但是不需要拦截我们的静态资源，使用前缀后我们直接排除这个前缀的/res/`**`  就可以了

### webjar(了解)

可用jar方式添加css，js等资源文件，自动映射 /[webjars](http://localhost:8080/webjars/jquery/3.5.1/jquery.js)/**

https://www.webjars.org/

例如，添加jquery

```xml
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.5.1</version>
        </dependency>
```

访问地址：[http://localhost:8080/webjars/**jquery/3.5.1/jquery.js**](http://localhost:8080/webjars/jquery/3.5.1/jquery.js)   后面地址要按照依赖里面的包路径

### 欢迎页支持

我们可以把我们的欢迎页放到静态资源路径下，这样访问我们的页面默认就会到这个欢迎页

> 这里的静态路径指的是默认的在那四个路径`/static`，`/public`,`/resources`, `/META-INF/resources`，如果我们指定了静态路径那么就用我们的路径作为静态路径，这4个会生效

- 静态资源路径下 放index.html

- - 可以配置静态资源路径
  - 但是不可以配置静态资源的访问前缀。否则导致 index.html不能被默认访问

```yml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致welcome page功能失效

  resources:
    static-locations: [classpath:/haha/]
```

- controller能处理/index

### 自定义 `Favicon`

favicon.ico 放在静态资源目录下即可

```yml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致 Favicon 功能失效
```

### 【源码】静态资源配置原理

- SpringBoot启动默认加载  xxxAutoConfiguration 类（自动配置类）
- SpringMVC功能的自动配置类在 WebMvcAutoConfiguration，生效

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {}
```

- 给容器中配了什么。

```java
	@Configuration(proxyBeanMethods = false)
	@Import(EnableWebMvcConfiguration.class)
	@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
	@Order(0)
	public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {}
```

- 配置文件的相关属性和xxx进行了绑定。WebMvcProperties==**spring.mvc**前缀、ResourceProperties==**spring.resources**前缀

#### 1、配置类只有一个有参构造器

```java
	//有参构造器所有参数的值都会从容器中确定
//ResourceProperties resourceProperties；获取和spring.resources绑定的所有的值的对象
//WebMvcProperties mvcProperties 获取和spring.mvc绑定的所有的值的对象
//ListableBeanFactory beanFactory Spring的beanFactory
//HttpMessageConverters 找到所有的HttpMessageConverters
//ResourceHandlerRegistrationCustomizer 找到 资源处理器的自定义器。=========
//DispatcherServletPath  
//ServletRegistrationBean   给应用注册Servlet、Filter....
	public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties, WebMvcProperties mvcProperties,
				ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
				ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
				ObjectProvider<DispatcherServletPath> dispatcherServletPath,
				ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
			this.resourceProperties = resourceProperties;
			this.mvcProperties = mvcProperties;
			this.beanFactory = beanFactory;
			this.messageConvertersProvider = messageConvertersProvider;
			this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
			this.dispatcherServletPath = dispatcherServletPath;
			this.servletRegistrations = servletRegistrations;
		}
```

#### 2、资源处理的默认规则

```java
@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
			//webjars的规则
            if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
            
            //这里可以设置一个缓存时间
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}
```

```yaml
spring:
#  mvc:
#    static-path-pattern: /res/**

  resources:
    add-mappings: false   禁用所有静态资源规则
```

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
	
    //ResourceProperties这里有几个默认值，就是这4个，也就是说我们可以使用自定义去替换这4个
	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };

	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
```

#### 3、欢迎页的处理规则

```java
	HandlerMapping：处理器映射。保存了每一个Handler能处理哪些请求。	

	@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
			welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
			return welcomePageHandlerMapping;
		}

	WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
			ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {
		if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {
            //要用欢迎页功能，必须是/**，这是底层写死的。所以我们使用前缀访问没有用
			logger.info("Adding welcome page: " + welcomePage.get());
			setRootViewName("forward:index.html");
		}
		else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
            // 调用Controller  /index
			logger.info("Adding welcome page template: index");
			setRootViewName("index");
		}
	}

```

# 请求参数处理

## 请求映射

### rest使用与原理

- @xxxMapping；
- Rest风格支持（*使用**HTTP**请求方式动词来表示对资源的操作*）

- - 以前：/getUser  *获取用户*    */deleteUser* *删除用户*   */editUser*  *修改用户*      */saveUser* *保存用户*
  - 现在： /user    GET-获取用户    DELETE-删除用户     PUT-修改用户     POST-保存用户
  - 核心Filter；HiddenHttpMethodFilter

- - - 用法： 表单method=post，隐藏域 _method=put
    - SpringBoot中手动开启

- - 扩展：如何把_method 这个名字换成我们自己喜欢的。

之前我们需要使用不同的名字作为请求的路径，一方面不规范，另外一方面起名麻烦，使用rest 后直接/user 的 type 设置成不同的请求类型就可以了

> 另外我们可以直接使用@GetMapping("/user")、@PostMapping("/user") 。其实就相当于RequestMapping(value = "/user",method = RequestMethod.GET)，更加简便了。以后都使用这种

```java

@GetMapping("/user")
//@RequestMapping(value = "/user",method = RequestMethod.GET)
public String getUser(){
    return "GET-张三";
}

@PostMapping("/user")
//@RequestMapping(value = "/user",method = RequestMethod.POST)
public String saveUser(){
    return "POST-张三";
}

@PutMapping("/user")
//@RequestMapping(value = "/user",method = RequestMethod.PUT)
public String putUser(){
    return "PUT-张三";
}

@DeleteMapping("/user")
//@RequestMapping(value = "/user",method = RequestMethod.DELETE)
public String deleteUser(){
    return "DELETE-张三";
}


```





#### 表单无法提交put和delete的问题

因为表单提交只能使用post和get 请求，那么如果我们想要发put 和delete请求怎么办呢？ 此时springboot 已经给我们通过自动配置好了一个拦截转换，将我们原生的request 进行封装

- **用法**
  - 1.开启页面表单的Rest功能
  - 2.页面 form的属性method=post，使用隐藏域 _method=put、delete等（如果直接get或post，无需隐藏域）

```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true   #开启页面表单的Rest功能
```

```html

<form action="/user" method="get">
    <input value="REST-GET提交" type="submit" />
</form>

<form action="/user" method="post">
    <input value="REST-POST提交" type="submit" />
</form>

<form action="/user" method="post">
    <input name="_method" type="hidden" value="DELETE"/>
    <input value="REST-DELETE 提交" type="submit"/>
</form>

<form action="/user" method="post">
    <input name="_method" type="hidden" value="PUT" />
    <input value="REST-PUT提交"type="submit" />
<form>

```



通过自动配置我们可以看到给我们配了一个这个OrderedHiddenHttpMethodFilter（），参数绑定是根绝这个属性的开启，如果开启了才放。（默认是关闭的），所以这里就是我们说的第一步，首先要在配置文件中开启这个

![image-20220312191638064](https://image.imxyu.cn/file/image-20220312191638064.png)

接下来我们点进去这个过滤器看一下，继承了父类的hiddentHttpMethodFilter

![image-20220312191848712](https://image.imxyu.cn/file/image-20220312191848712.png)

看一下父类的过滤的方法doFilterInternal（）

首先会判断如果是post请求才会进行requst的转换。

1、接下来首先获取到了_method 这个中的值

2、然后会将其转换成大写，也就是说这里如果我们写成小写的delete 也是可以的，因为这里会进行自动转换

![image-20220312191913998](https://image.imxyu.cn/file/image-20220312191913998.png)



![image-20220312192032989](https://image.imxyu.cn/file/image-20220312192032989.png)

如果是这四种类型的话才将其转换，将原生的request 请求进行一层包装，包装成HttpMethodRequestWrapper

![image-20220312192121034](https://image.imxyu.cn/file/image-20220312192121034.png)

将我们的_method 中的值转换成大写后人传入到这个包装类中，使用我们的值作为request的method。这样controller 中就能匹配到我们的delete和put请求了，通过controller 中的request参数拿到的request 也是我们通过包装后的HttpMethodRequestWrapper

![image-20220312192338423](https://image.imxyu.cn/file/image-20220312192338423.png)



Rest原理（表单提交要使用REST的时候）

- 表单提交会带上**_method=PUT**
- **请求过来被**HiddenHttpMethodFilter拦截

- - 请求是否正常，并且是POST

- - - 获取到**_method**的值。
    - 兼容以下请求；**PUT**.**DELETE**.**PATCH**
    - **原生request（post），包装模式requesWrapper重写了getMethod方法，返回的是传入的值。**
    - 过滤器链放行的时候用wrapper。以后的方法调用getMethod是调用requesWrapper的。



**Rest使用客户端工具，**

- 如PostMan直接发送Put、delete等方式请求，无需Filter（因为源码中写的只有请求是post的时候才会进入这个过滤器的拦截判断逻辑）。

或者以后我们使用前后端分离的话，前端能直接发delete和put 请求，我们就不会在yml 中配置那个参数了

这里仅限于表单发送的问题。

##### 自定义修改隐藏域_method 

上面我们讲到了自动配置的底层会帮我们放一个OrderedHiddenHttpMethodFilter的过滤器，帮我们实现的这个请求转换，并且是当我们容器中没有这个类型的过滤器的时候才会帮我们放置，那么我们可以自定义一个这个过滤放到容器中，这样就不会给我们放了

![image-20220312193149017](https://image.imxyu.cn/file/image-20220312193149017.png)

并且我们观察到这个hiddenHttpMethodFilter 中的这个DEFAULT_METHOD_PARAM属性虽然是静态的final， 但是真正用的是这个methodParam, 并且给我们提供了一个set 方法去修改这个methodParam，那么我们就可以通过这个方法进行设置成我们自己想要的隐藏域的名字

![image-20220312193400759](https://image.imxyu.cn/file/image-20220312193400759.png)



在配置类中创建一个，并且通过setMethodParam设置我们想要的值

```java

@Configuration(proxyBeanMethods = false)
public class WebConfig{
    //自定义filter
    @Bean
    public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
        methodFilter.setMethodParam("_m");
        return methodFilter;
    }    
}

```



将`\_method`改成我们自定义的`_m`。

```html

<form action="/user" method="post">
    <input name="_m" type="hidden" value="DELETE"/>
    <input value="REST-DELETE 提交" type="submit"/>
</form>


```

### 【源码】请求映射原理

我们来发送一个rest请求来看一下，这个请求是怎么被找到的？

```
测试REST风格；
<form action="/user" method="get">
    <input value="REST-GET 提交" type="submit"/>
</form>
```

dispatcherServlet继承图



![image.png](https://image.imxyu.cn/file/1603181171918-b8acfb93-4914-4208-9943-b37610e93864.png)



SpringMVC功能分析都从 org.springframework.web.servlet.DispatcherServlet-》doDispatch（），最终都要调用子类继承的这个方法

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// 找到当前请求使用哪个Handler（Controller的方法）处理
				mappedHandler = getHandler(processedRequest);
                
                //HandlerMapping：处理器映射。/xxx->>xxxx
```

主要这个方法来帮我们找到的对应的handler处理请求

**mappedHandler = getHandler(processedRequest);**

![image-20220313095345479](https://image.imxyu.cn/file/image-20220313095345479.png)

会按照这5个顺序进行匹配，看哪个能处理就返回哪个

![img](https://image.imxyu.cn/file/1603181460034-ba25f3c0-9cfd-4432-8949-3d1dd88d8b12.png)

这里我们还可以看到springboot 自动配置给我们注册的欢迎页的WelcomePagehandlerMapping ，

对于所有的方法上的请求都会通过**RequestMappingHandlerMapping**：这个处理器映射保存了所有@RequestMapping 和handler的映射规则。

> 项目启动的时候就会创建了这个对象RequestMappingHandlerMapping，保存了所有的方法和映射的请求

下面就是RequestMappingHandlerMapping中保存的所有信息，其中就有我们的路径和方法的映射map（key = 请求路径  ，value =方法名。来进行保存的）

![img](https://image.imxyu.cn/file/1603181662070-9e526de8-fd78-4a02-9410-728f059d6aef.png)

我们继续点进这个方法看一下

![image-20220313095400249](https://image.imxyu.cn/file/image-20220313095400249.png)

继续看

![image-20220313095512447](https://image.imxyu.cn/file/image-20220313095512447.png)

可以看到这里面就是获取保存到**RequestMappingHandlerMapping**中**registry**中所有 url ,找到我们要找的/user 路径。可以看到这里找到了4条，

然后后面会进行筛选看哪一条满足，把满足的那一条返回。如果有多条满足下面会抛出异常。

![image-20220313100204953](https://image.imxyu.cn/file/image-20220313100204953.png)



### 【源码】欢迎页的请求映射原理

我们刚刚看到handlerMapping 中有一个welcomeHandlerMapping，就是用来处理我们的欢迎页的，我们直接访问http://localhost:8080/一下看看是怎么处理的



首先会找到所有的HandlerMapping ，按照第一个开始处理，第一个RequestHandlerMapping ，返回了null ，因为这里面是专门处理我们写在方法上的请求的，所以这个处理不了，于是返回了null

![image-20220313100713526](https://image.imxyu.cn/file/image-20220313100713526.png)

接下来我们看到欢迎页的handlerMapping 可以处理，然后帮我们返回了一个处理的handler，这个是spring自动配置帮我们配的一个controller ，并且返回的view 是转发到Index.html

![image-20220313100842200](https://image.imxyu.cn/file/image-20220313100842200.png)

总结：

- SpringBoot自动配置欢迎页的 WelcomePageHandlerMapping 。访问 /能访问到index.html；
- SpringBoot自动配置了默认 的 RequestMappingHandlerMapping
- 请求进来，挨个尝试所有的HandlerMapping看是否有请求信息。
- - 如果有就找到这个请求对应的handler
  - 如果没有就是下一个 HandlerMapping
- 我们需要一些自定义的映射处理，我们也可以自己给容器中放**HandlerMapping**。自定义 **HandlerMapping**



这个就是自动配置类帮我们配置的处理方法上的请求的处理器RequestMappingHandlerMapping

![image-20220313101435415](https://image.imxyu.cn/file/image-20220313101435415.png)

这里详细的源码请看总结的大厂学院的spring mvc 的源码部分即可

## 普通参数与基本注解

### 请求处理常用注解：

#### 熟悉的几个注解

@PathVariable 路径变量  ， 可以指定，也可以用一个Map获取所有的
@RequestHeader 获取请求头，可以指定，也可以用一个Map获取所有的
@RequestParam 获取请求参数（指问号后的参数，url?a=1&b=2）
@CookieValue 获取Cookie值，就是浏览器过来携带的值   ,   可以指定，也可以用一个Cookie获取所有的
@RequestAttribute 获取request域中的属性
@RequestBody 获取请求体[POST] , 一般只有post请求才有请求体，get 一般都在地址栏上拼接
@MatrixVariable 矩阵变量
@ModelAttribute

```java
@RestController
public class ParameterTestController {


    //  car/2/owner/zhangsan
    @GetMapping("/car/{id}/owner/{username}")
    public Map<String,Object> getCar(@PathVariable("id") Integer id,
                                     @PathVariable("username") String name,
                                     @PathVariable Map<String,String> pv,
                                     @RequestHeader("User-Agent") String userAgent,
                                     @RequestHeader Map<String,String> header,
                                     @RequestParam("age") Integer age,
                                     @RequestParam("inters") List<String> inters,
                                     @RequestParam Map<String,String> params,
                                     @CookieValue("_ga") String _ga,
                                     @CookieValue("_ga") Cookie cookie){


        Map<String,Object> map = new HashMap<>();

//        map.put("id",id);
//        map.put("name",name);
//        map.put("pv",pv);
//        map.put("userAgent",userAgent);
//        map.put("headers",header);
        map.put("age",age);
        map.put("inters",inters);
        map.put("params",params);
        map.put("_ga",_ga);
        System.out.println(cookie.getName()+"===>"+cookie.getValue());
        return map;
    }

		// 这里使用requestbody 获取post表单中的请求体内容
    @PostMapping("/save")
    public Map postMethod(@RequestBody String content){
        Map<String,Object> map = new HashMap<>();
        map.put("content",content);
        return map;
    }

}
```

#### @RequestAttribute 获取request域中的属性

```java

@Controller
public class RequestController {

    @GetMapping("/goto")
    public String goToPage(HttpServletRequest request){

        request.setAttribute("msg","成功了...");
        request.setAttribute("code",200);
        return "forward:/success";  //转发到  /success请求, 因为只有转发才能携带request 请求中的数据
    }

    @GetMapping("/params")
    public String testParam(Map<String,Object> map,
                            Model model,
                            HttpServletRequest request,
                            HttpServletResponse response){
        map.put("hello","world666");
        model.addAttribute("world","hello666");
        request.setAttribute("message","HelloWorld");

        Cookie cookie = new Cookie("c1","v1");
        response.addCookie(cookie);
        return "forward:/success";
    }

    ///<-----------------主角@RequestAttribute在这个方法，获取来request 请求域中的属性值
    // model 和map 这几个参数我们先不用关注，后面会讲到
    @ResponseBody
    @GetMapping("/success")
    public Map success(@RequestAttribute(value = "msg",required = false) String msg,
                       @RequestAttribute(value = "code",required = false)Integer code,
                       HttpServletRequest request){
        Object msg1 = request.getAttribute("msg");

        Map<String,Object> map = new HashMap<>();
        Object hello = request.getAttribute("hello");
        Object world = request.getAttribute("world");
        Object message = request.getAttribute("message");

        map.put("reqMethod_msg",msg1);
        map.put("annotation_msg",msg);
        map.put("hello",hello);
        map.put("world",world);
        map.put("message",message);

        return map;
    }
}


```

#### (自定义web组件)@MatrixVariable与UrlPathHelper

1、语法： 请求路径：/cars/sell;low=34;brand=byd,audi,yd

2、SpringBoot默认是禁用了矩阵变量的功能，会自动移除路径中的分号

3、手动开启：需要自定义一个UrlPath。对于路径的处理。UrlPathHelper的removeSemicolonContent设置为false，让其支持矩阵变量的。
矩阵变量必须有url路径变量才能被解析

**手动开启矩阵变量**的两种方式：



- 1、实现`WebMvcConfigurer`接口：

```java

@Configuration(proxyBeanMethods = false)
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {

        UrlPathHelper urlPathHelper = new UrlPathHelper();
        // 不移除；后面的内容。矩阵变量功能就可以生效
        urlPathHelper.setRemoveSemicolonContent(false);
        configurer.setUrlPathHelper(urlPathHelper);
    }
}


```

- 2、创建返回`WebMvcConfigurer`Bean：

```java

@Configuration(proxyBeanMethods = false)
public class WebConfig{
    @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
                        @Override
            public void configurePathMatch(PathMatchConfigurer configurer) {
                UrlPathHelper urlPathHelper = new UrlPathHelper();
                // 不移除；后面的内容。矩阵变量功能就可以生效
                urlPathHelper.setRemoveSemicolonContent(false);
                configurer.setUrlPathHelper(urlPathHelper);
            }
        }
    }
}


```

**`@MatrixVariable`的用例**

```java

@RestController
public class ParameterTestController {

    ///cars/sell;low=34;brand=byd,audi,yd
    @GetMapping("/cars/{path}")
    public Map carsSell(@MatrixVariable("low") Integer low,
                        @MatrixVariable("brand") List<String> brand,
                        @PathVariable("path") String path){
        Map<String,Object> map = new HashMap<>();

        map.put("low",low);
        map.put("brand",brand);
        map.put("path",path);
        return map;
    }

    // /boss/1;age=20/2;age=10

    @GetMapping("/boss/{bossId}/{empId}")
    public Map boss(@MatrixVariable(value = "age",pathVar = "bossId") Integer bossAge,
                    @MatrixVariable(value = "age",pathVar = "empId") Integer empAge){
        Map<String,Object> map = new HashMap<>();

        map.put("bossAge",bossAge);
        map.put("empAge",empAge);
        return map;

    }

}


```

> 使用@MatrixVariable 可以用来获取路径中的多个值，并且划分清晰。如果路径中需要传递多个值可以用这种方式
>
> 注意这里的路径参数中必须要用{ }这样来接收



#### @MatrixVariable底层自动移除 ; 分号的原理

我们可以在WebMvcAutoConfiguration这个自动配置类中找到一个子类，这个类实现了这个WebMvcConfigurer 接口，我们继续往下看

![image-20220312202919850](https://image.imxyu.cn/file/image-20220312202919850.png)

并且这个类里面有一个重写的方法，这个方法中会有这个urlHelper ，这个就是帮我们处理路径的

![image-20220312203245545](https://image.imxyu.cn/file/image-20220312203245545.png)

点开这个类发现有这个属性，默认是true ,也就是移除路径中的分号

![image-20220312202623899](https://image.imxyu.cn/file/image-20220312202623899.png)

因此官方告诉我们了如果我们想要自定义拦截器、view controllers 我们可以通过@Configuration+实现WebMvcConfigurer 自定义规则，同样我们也可以自定义一个这个WebMvcConfigurer组件注册到容器中。上面的两种方式就是我们的解决方案（这里我们需要自定义组件之后把setUrlPathHelper这个参数设置成false即可）

![image-20220312203632457](https://image.imxyu.cn/file/image-20220312203632457.png)



### 【源码】请求处理参数解析

接下来我们看一下源码这个参数值是怎么处理的。我们通过这个地址访问一下看看

http://localhost:8080/car/3/owner/lisi?age=18&inters=basketball&inters=game

首先在第一步的时候找到了相应的HandlerMapping, 可以看到找到了我们ParameterTestController 的getCar 请求能处理我们的方法

接下来会寻找handlerAdapter ，这个相当于一个强大反射工具，可以帮我们在参数上进行处理，我们点进去看看

![image-20220313113431568](https://image.imxyu.cn/file/image-20220313113431568.png)

可以看到这里面也有很多的handlerAdapter ，这里会循环遍历所有的adapter，看哪个能处理我们的这个handler

此时第一个就能处理，返回了第一个adapter ，RequestMappingHandlerAdapter

![image-20220313113757553](https://image.imxyu.cn/file/image-20220313113757553.png)



接下来会调用这个adpter， 处理这个handler

![image-20220313113949804](https://image.imxyu.cn/file/image-20220313113949804.png)

![image-20220313114009098](https://image.imxyu.cn/file/image-20220313114009098.png)

在这个方法中继续调用

![image-20220313115730863](https://image.imxyu.cn/file/image-20220313115730863.png)

在这个方法中有两个重要的解析器，参数值解析器和返回值解析器，我们可以看到这个参数值解析器有26个，这26个包含了所有能处理的参数的类型。这里是把这26都放到了这个包装类型中invocableMethod , 后面参数解析的时候会进行挨个判断

![image-20220313115815756](https://image.imxyu.cn/file/image-20220313115815756.png)

接下来下面有一个方法这个，就是调用处理的过程

![image-20220313120150161](https://image.imxyu.cn/file/image-20220313120150161.png)

首先会处理我们的参数

![image-20220313120221624](https://image.imxyu.cn/file/image-20220313120221624.png)

继续看这个方法，就是处理我们方法中参数上的类型赋值的getMethodArgumentValues

![image-20220313120240983](https://image.imxyu.cn/file/image-20220313120240983.png)

首先得到了方法上的十个参数，然后创建了一个value的数组，用来保存这十个参数所对应的值

![image-20220313120343123](https://image.imxyu.cn/file/image-20220313120343123.png)

#### 查找所有参数对应支持的参数解析器

下面for循环遍历每一个参数，去确定每一个参数的值

首先会调用所有的参数解析器，看哪个参数解析器能处理这个参数。如果都不能的话就会抛出异常

``` 
this.resolvers.supportsParameter(parameter) 我们点进去这个方法看一下
```

![image-20220313120656192](https://image.imxyu.cn/file/image-20220313120656192.png)

可以看到第一个处理的参数是@PathVarible 注解，会遍历所有的参数解析器调用他们的support 方法，看哪个能解析这个参数。在这个方法中首先会从缓存中拿，也就是说如果我们之前处理过 会把相应的结果保存到（按照key---参数,  value----参数值解析器）的形式保存到map中

![image-20220313123117409](https://image.imxyu.cn/file/image-20220313123117409.png)

对于参数解析器接口一共就这两个方法，一个是看自己是否能支持处理这个请求support，另一个是进行处理的方法resolve

![image-20220313120859461](https://image.imxyu.cn/file/image-20220313120859461.png)

首先来到我们的第一个参数解析器requestParam ，这个参数解析器判断很简单就是看你有没有标注@RequestParam 注解，如果标注了就return true

![image-20220313122936844](https://image.imxyu.cn/file/image-20220313122936844.png)

这样循环之后就找到了第一个参数能处理的参数解析器pathvariableMethodArugmentResolver ，并且把它放到缓存中，之后就不用判断了直接从缓存中取就可以了

![image-20220313123217277](https://image.imxyu.cn/file/image-20220313123217277.png)

#### 调用参数解析器处理得到值

找到对应的参数解析器之后，就可以调用参数解析器的解析方法了

![image-20220313123657341](https://image.imxyu.cn/file/image-20220313123657341.png)

第一步还是获取参数解析器

![image-20220313123731112](https://image.imxyu.cn/file/image-20220313123731112.png)

还是这个方法，确定参数解析器的方法。但是此时因为上面判断support的时候就已经把对应的参数解析器放到缓存中了，所以这里第一步就可以从缓存中拿到值了

![image-20220313123800145](https://image.imxyu.cn/file/image-20220313123800145.png)

下面就是调用参数解析的流程，拿到参数解析器后调用参数解析器的resolve方法

![image-20220313124003945](https://image.imxyu.cn/file/image-20220313124003945.png)



得到我们要pathvarible 获取的的值，进入路径中寻找这个值

![image-20220313124123337](https://image.imxyu.cn/file/image-20220313124123337.png)



可以看到这里面保存了路径中的所有值，这个就是我们之前说过UrlHelper 帮我们干的，把请求的路径保存了下来，这里直接通过get 就可以获取到了

![image-20220313124155076](https://image.imxyu.cn/file/image-20220313124155076.png)

此时这个for循环中的第一个参数就解析完了，得到值3，保存到了这个value 数组中

![image-20220313124457315](https://image.imxyu.cn/file/image-20220313124457315.png)



其他的所有参数都是会进行这个流程

1、在26个参数解析器中寻找支持的参数解析器，并且保存到缓存中（这样下次就省去for循环26个解析器判断的流程了，直接从缓存中拿对应参数的解析器就好）

​																		-》2、利用参数解析器的解析方法-》3、得到的值保存到这个value数组中

至此所有的参数就解析完毕了



这26个参数解析器就在这，所有我们能在方法上写的参数都是因为有一个参数解析器能support支持。

![image-20220313124823279](https://image.imxyu.cn/file/image-20220313124823279.png)

### Servlet API：

请求参数上不仅能放我们的那几个常用的注解，同时还可以放我们的Servlet API：

- WebRequest
- ServletRequest
- MultipartRequest
- HttpSession
- javax.servlet.http.PushBuilder
- Principal
- InputStream
- Reader
- HttpMethod
- Locale
- TimeZone
- ZoneId

#### selevet APi 参数解析器源码

再比如说我们上面的这个/goto 路径的请求

![image-20220313132640595](https://image.imxyu.cn/file/image-20220313132640595.png)

对于这个请求，使用的就是这个参数解析器，ServletRequestMethodArgumentResolver 这里面会判断参数中是否有下面这几个，而我们的request 就在里面

![image-20220313133113075](https://image.imxyu.cn/file/image-20220313133113075.png)



然后在这个参数解析器的resolve处理逻辑中，判断如果是servletRequest 的参数的话，就会得到原生的request对象然后帮我们注入进去就行

![image-20220313133304446](https://image.imxyu.cn/file/image-20220313133304446.png)

### 复杂参数

同时支持我们放一些复杂的参数

复杂参数：

**Map**

**Model**（map、model里面的数据会被放在request的请求域 request.setAttribute）

Errors/BindingResult

**RedirectAttributes**（ 重定向携带数据）

**ServletResponse**（response）

SessionStatus

UriComponentsBuilder

ServletUriComponentsBuilder



这里我们主要测试一个map, model 和response。 通过在/params 这个路径处理的方法上放值，然后将这个请求转发给/suceess 路径的方法中，因为是转发所以此时request 也会传递过去，然后我们通过第二个方法获取到传过来的request ，就能取到里面设置的值（就是通过model 和map 设置到request域中的）

```java

@GetMapping("/params")
public String testParam(Map<String,Object> map,
                        Model model,
                        HttpServletRequest request,
                        HttpServletResponse response){
    //下面三位都是可以给request域中放数据
    map.put("hello","world666");
    model.addAttribute("world","hello666");
    request.setAttribute("message","HelloWorld");

    Cookie cookie = new Cookie("c1","v1");
    response.addCookie(cookie);
    return "forward:/success";
}

@ResponseBody
@GetMapping("/success")
public Map success(@RequestAttribute(value = "msg",required = false) String msg,
                   @RequestAttribute(value = "code",required = false)Integer code,
                   HttpServletRequest request){
    Object msg1 = request.getAttribute("msg");

    Map<String,Object> map = new HashMap<>();
    Object hello = request.getAttribute("hello");//得出testParam方法赋予的值 world666
    Object world = request.getAttribute("world");//得出testParam方法赋予的值 hello666
    Object message = request.getAttribute("message");//得出testParam方法赋予的值 HelloWorld

    map.put("reqMethod_msg",msg1);
    map.put("annotation_msg",msg);
    map.put("hello",hello);
    map.put("world",world);
    map.put("message",message);

    return map;
}


```

#### 【源码】model、map放到request中

接下来我们看一下这个model、 map 是怎么被一步步放到request中的？

1、Map类型参数

首先同样的需要先找到参数解析器，第一个参数是Map, 我们看到在26个参数解析器中找到了Map的参数解析器

![image-20220313142943309](https://image.imxyu.cn/file/image-20220313142943309.png)



接下来调用这个参数解析器的resolve 方法

![image-20220313143142360](https://image.imxyu.cn/file/image-20220313143142360.png)

![image-20220313143210312](https://image.imxyu.cn/file/image-20220313143210312.png)

可以看到返回的这个model 就是这个BindingAwareModelMap

![image-20220313143243125](https://image.imxyu.cn/file/image-20220313143243125.png)

![image-20220313143247716](https://image.imxyu.cn/file/image-20220313143247716.png)

2、Model类型参数

接下来我们看一下这个Model类型的参数 。

可以看到model 类型的参数解析器是ModelMethodProcessor 。接下来我们看一下这个参数的处理流程

![image-20220313143422573](https://image.imxyu.cn/file/image-20220313143422573.png)

可以看到这个参数解析器解析的流程也是返回getModel()，和上面的Map是一样的。都是返回这个BindingAwareModelMap

![image-20220313143901441](https://image.imxyu.cn/file/image-20220313143901441.png)

并且我们注意到的是这两个类型得到的都是同一个BindingAwareModelMap对象 ，对象地址都是这个6678

![image-20220313144054302](https://image.imxyu.cn/file/image-20220313144054302.png)

此时这个参数解析的过程就完成了，得到了所有参数上的值，然后就可以反射调用这个方法了，下面这个就是反射调用方法的流程

并且这个BindingAwareModelMap对象中保存了model 和map类型的这两个值，无论是模型还是视图都在这个对象中

![image-20220313151633658](https://image.imxyu.cn/file/image-20220313151633658.png)



![image-20220313144246520](https://image.imxyu.cn/file/image-20220313144246520.png)

并且反射调用完方法之后会把我们方法的返回值返回给我们

![image-20220313144446421](https://image.imxyu.cn/file/image-20220313144446421.png)



拿到方法的返回值，然后处理返回值，这里拿到了我们的forward:/success

![image-20220313144657173](https://image.imxyu.cn/file/image-20220313144657173.png)

这个方法就会将我们的返回值放到modelAndView对象的view 中

![image-20220313145628577](https://image.imxyu.cn/file/image-20220313145628577.png)

此时mvContainer 中就有了model 和view

![image-20220313152128445](https://image.imxyu.cn/file/image-20220313152128445.png)

这个方法就结束了，然后返回modelAndView

![image-20220313145929601](https://image.imxyu.cn/file/image-20220313145929601.png)



**接下来看看**`Map<String,Object> map`与`Model model`值是如何做到用`request.getAttribute()`获取的。

所有的数据都放在 **ModelAndView**包含要去的页面地址View，还包含Model数据。

调用render（）方法渲染的时候，这里使用InternalResourceView的render() 方法进行渲染得到我们的页面的时候

```java

public class InternalResourceView extends AbstractUrlBasedView {
    
 	@Override//该方法在AbstractView，AbstractUrlBasedView继承了AbstractView
	public void render(@Nullable Map<String, ?> model, HttpServletRequest request,
			HttpServletResponse response) throws Exception {
		
        ...
        
		Map<String, Object> mergedModel = createMergedOutputModel(model, request, response);
		prepareResponse(request, response);
        
        //看下一个方法实现
		renderMergedOutputModel(mergedModel, getRequestToExpose(request), response);
	}
    
    @Override
	protected void renderMergedOutputModel(
			Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {

		// Expose the model object as request attributes.
        // 暴露模型作为请求域属性
		exposeModelAsRequestAttributes(model, request);//<---重点

		// Expose helpers as request attributes, if any.
		exposeHelpers(request);

		// Determine the path for the request dispatcher.
		String dispatcherPath = prepareForRendering(request, response);

		// Obtain a RequestDispatcher for the target resource (typically a JSP).
		RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);
		
        ...
	}
    
    //该方法在AbstractView，AbstractUrlBasedView继承了AbstractView
    protected void exposeModelAsRequestAttributes(Map<String, Object> model,
			HttpServletRequest request) throws Exception {

        //通过这里for循环把model 中的值设置到request属性中
		model.forEach((name, value) -> {
			if (value != null) {
				request.setAttribute(name, value);
			}
			else {
				request.removeAttribute(name);
			}
		});
	}
    
}


```

`exposeModelAsRequestAttributes`方法看出，`Map<String,Object> map`，`Model model`这两种类型数据可以给request域中放数据，用`request.getAttribute()`获取。



## 自定义类型pojo的参数绑定

参数上还可以传我们自定义的pojo 对象，也会帮我们绑定

```java
@RestController
public class ParameterTestController {

    /**
     * 数据绑定：页面提交的请求数据（GET、POST）都可以和对象属性进行绑定
     * @param person
     * @return
     */
    @PostMapping("/saveuser")
    public Person saveuser(Person person){
        return person;
    }
```

![image-20220313162022654](https://image.imxyu.cn/file/image-20220313162022654.png)

### pojo绑定源码

这个自定义的pojo 类型的参数绑定使用的是这个参数解析器**ServletModelAttributeMethodProcessor**

```java

public class ServletModelAttributeMethodProcessor extends ModelAttributeMethodProcessor {
	
    @Override//本方法在ModelAttributeMethodProcessor类，
	public boolean supportsParameter(MethodParameter parameter) {
		return (parameter.hasParameterAnnotation(ModelAttribute.class) ||
				(this.annotationNotRequired && !BeanUtils.isSimpleProperty(parameter.getParameterType())));
	}

	@Override
	@Nullable//本方法在ModelAttributeMethodProcessor类，
	public final Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

		...

		String name = ModelFactory.getNameForParameter(parameter);
		ModelAttribute ann = parameter.getParameterAnnotation(ModelAttribute.class);
		if (ann != null) {
			mavContainer.setBinding(name, ann.binding());
		}

		Object attribute = null;
		BindingResult bindingResult = null;

		if (mavContainer.containsAttribute(name)) {
			attribute = mavContainer.getModel().get(name);
		}
		else {
			// Create attribute instance
			try {
				attribute = createAttribute(name, parameter, binderFactory, webRequest);
			}
			catch (BindException ex) {
				...
			}
		}

		if (bindingResult == null) {
			// Bean property binding and validation;
			// skipped in case of binding failure on construction.
			WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);
			if (binder.getTarget() != null) {
				if (!mavContainer.isBindingDisabled(name)) {
                    //web数据绑定器，将请求参数的值绑定到指定的JavaBean里面**
					bindRequestParameters(binder, webRequest);
				}
				validateIfApplicable(binder, parameter);
				if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
					throw new BindException(binder.getBindingResult());
				}
			}
			// Value type adaptation, also covering java.util.Optional
			if (!parameter.getParameterType().isInstance(attribute)) {
				attribute = binder.convertIfNecessary(binder.getTarget(), parameter.getParameterType(), parameter);
			}
			bindingResult = binder.getBindingResult();
		}

		// Add resolved attribute and BindingResult at the end of the model
		Map<String, Object> bindingResultModel = bindingResult.getModel();
		mavContainer.removeAttributes(bindingResultModel);
		mavContainer.addAllAttributes(bindingResultModel);

		return attribute;
	}
}


```

首先会调用这个方法去创建一个要绑定的对象，放到这个binder中

![image-20220313155756100](https://image.imxyu.cn/file/image-20220313155756100.png)





WebDataBinder 利用它里面的 Converters 将请求数据转成指定的数据类型。再次封装到JavaBean中

因为我们的值是通过http 超文本传输的，因此都是字符串，会通过这个WebDataBinder 中的converters 进行类型转换，比如说我们传的年龄18 通过http 其实是字符串类型，使用这里面的字符串转Integer 类型的converter 就可以进行转换。

![image-20220313155357669](https://image.imxyu.cn/file/image-20220313155357669.png)

当这个方法被调用完之后所有的属性就被绑定好了

![image-20220313160422965](https://image.imxyu.cn/file/image-20220313160422965.png)



![image-20220313160452165](https://image.imxyu.cn/file/image-20220313160452165.png)



在bind方法中包含了所有从浏览器中拿到的这些字符串，已经通过key,value的形式将他们封装到了这个mpvs中。在这里面调用doBind(mpvs)方法就将所有的值绑定到了binder对象上 。

![image-20220313160517652](https://image.imxyu.cn/file/image-20220313160517652.png)



接下来就是调用反射赋值的过程，其中里面值类型进行转换的过程我们简单看一下。可以看到这里有需要的类型值，然后经过这个conversionService中的converter 进行

![image-20220313161104135](https://image.imxyu.cn/file/image-20220313161104135.png)

总结：

**WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);**

**WebDataBinder :web数据绑定器，将请求参数的值绑定到指定的JavaBean里面**

**WebDataBinder 利用它里面的 Converters 将请求数据转成指定的数据类型。再次封装到JavaBean中**



**GenericConversionService：在设置每一个值的时候，找它里面的所有converter那个可以将这个数据类型（request带来参数的字符串）转换到指定的类型（JavaBean -- Integer）**

**byte -- > file**



未来我们可以给WebDataBinder里面放自己的自定义的Converter；

**private static final class** StringToNumber<T **extends** Number> **implements** Converter<String, T>

### 【*自定义web组件】针对Pojo类型自定义converter封装属性

现在我们有一个需求：

​		就是不使用之前的这种传递多个参数的封装宠物pet.name,pet.age ，觉得太麻烦，想直接传一个参数为pet字符串，通过逗号分隔，那么就需要我们自定义converter

![image-20220313163145105](https://image.imxyu.cn/file/image-20220313163145105.png)

如何自定义WEB中的组件呢？ 之前我们说过springboot给我们提供了两种方式（在矩阵变量那里）



![image-20220313164359165](https://image.imxyu.cn/file/image-20220313164359165.png)

1、让配置类实现webMvcConfigurer接口，去覆盖里面的方法。

2、@Bean直接创建一个WebMvcConfigurer的对象返回 （我们使用这种）

可以看到这个方法中有很多方法支持我们自定义一些配置

![image-20220313164756673](https://image.imxyu.cn/file/image-20220313164756673.png)

我们这里直接创建了一个webMvcConfigurer组件，然后调用addFormatters 给容器中添加一个转换器就可以了

```java
//1、WebMvcConfigurer定制化SpringMVC的功能
@Bean
public WebMvcConfigurer webMvcConfigurer(){
    return new WebMvcConfigurer() {

        //自定义了一个将string 类型转换成pet类型的c
        @Override
        public void addFormatters(FormatterRegistry registry) {
            registry.addConverter(new Converter<String, Pet>() {

                @Override
                public Pet convert(String source) {
                    // 啊猫,3
                    if(!StringUtils.isEmpty(source)){
                        Pet pet = new Pet();
                        String[] split = source.split(",");
                        pet.setName(split[0]);
                        pet.setAge(Integer.parseInt(split[1]));
                        return pet;
                    }
                    return null;
                }
            });
        }
    };
}
```

这个convter 也比较简单，我们只需要传入原类型和目标类型即可， 因为是http超文本中传递的，所以原类型是字符串类型，目标类型是我们的pet宠物类型

然后覆盖convert方法进行转换就可以了

![image-20220313164905153](https://image.imxyu.cn/file/image-20220313164905153.png)

> 当我们把自定义的converter添加到容器中后，就会找到从124-》125个转换器，此时给pojo进行赋值的时候，因为person中的pet类型，当我们传入类型为string，而要求的类型是pet的时候就会调用我们的这个converter转换器

# 数据响应与内容协商

![image-20220313180533907](https://image.imxyu.cn/file/image-20220313180533907.png)

## 响应json

jackson.jar+@ResponseBody

我们的springboot-starter-web 自动帮我们引入了json的场景，默认使用的是jackson

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
            
		引入上面web场景的时候自动引入了下面这个json场景
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-json</artifactId>
      <version>2.3.4.RELEASE</version>
      <scope>compile</scope>
    </dependency>

```

默认使用的是jackson

![image.png](https://image.imxyu.cn/file/1605151090728-f7c60e6f-d0c0-4541-bfa3-8cc805dfd5d6.png)



此时我们可以直接通过@ResponseBody注解来直接返回json 字符串

```java
 @ResponseBody  //利用返回值处理器里面的消息转换器进行处理
    @GetMapping(value = "/test/person")
    public Person getPerson(){
        Person person = new Person();
        person.setAge(28);
        person.setBirth(new Date());
        person.setUserName("zhangsan");
        return person;
    }
```

可以看到浏览器能直接得到返回的json数据（这里我用了jsonviewer解析json 的插件）

![image-20220313194332717](https://image.imxyu.cn/file/image-20220313194332717.png)

### 【源码】找到返回值处理器

请求参数那里我们就不看了。我们来看一下得到返回值后是如何处理的

![image-20220313182241873](https://image.imxyu.cn/file/image-20220313182241873.png)

通过我们的返回值的类型，去找返回值的处理器，我们点进这个方法看一下

![image-20220313182417549](https://image.imxyu.cn/file/image-20220313182417549.png)

会根据我们这个返回值类型去在这个方法中会循环15种返回值处理器，调用support方法判断看哪个能支持这种类型的返回值处理器

这里也就决定了我们一共能写多少种返回值

![image-20220313182930341](https://image.imxyu.cn/file/image-20220313182930341.png)

![image-20220313182455161](https://image.imxyu.cn/file/image-20220313182455161.png)

因为我们这里的返回值类型是Person类型的，并且我们标注了@ResponseBody 注解

因此我们在这个RequestResponseBodyMethodProcessor返回值处理器中找到了满足条件的

![image-20220313183102928](https://image.imxyu.cn/file/image-20220313183102928.png)

```java
//一共能写这么多种返回值的类型
ModelAndView
Model
View
ResponseEntity 
ResponseBodyEmitter
StreamingResponseBody
HttpEntity
HttpHeaders
Callable
DeferredResult
ListenableFuture
CompletionStage
WebAsyncTask
有 @ModelAttribute 且为对象类型的
@ResponseBody 注解 ---> RequestResponseBodyMethodProcessor；

```

找到之后调用返回值处理器的handleReturnValue方法

![image-20220313183149816](https://image.imxyu.cn/file/image-20220313183149816.png)



### 【源码】@Responsebody返回值处理器处理

可以看到返回值处理器一共就两个方法，一个是判断是否支持，一个是进行处理的过程

![image.png](https://image.imxyu.cn/file/1605151728659-68c8ce8a-1b2b-4ab0-b86d-c3a875184672.png)

接下来会进行放回值处理器进行处理

![image-20220313184825233](https://image.imxyu.cn/file/image-20220313184825233.png)



接下来进入writeWithMessageConverters这个核心方法，首先得到我们返回的值的类型，然后得到我们的目标类型

![image-20220313185031234](https://image.imxyu.cn/file/image-20220313185031234.png)



下面首先介绍一下内容协商的概念，浏览器默认会以请求头的方式告诉服务器他能接受什么样的内容类型

当我们浏览器发送一个请求的时候，会把能接受的请求也会在请求头上标注，然后携带过去。

可以看到这个请求能接受text/html,application/xhtml+xml,application/xml;q=0.9  这个数字的意思是优先级，优先给我返回这几个类型

image/avif,image/webp,image/apng,*/*;`*/*` q=0.8,   后面这个`*/*` 的意思是任意类型，也就是说其他任意类型给我都可以

![image-20220313185305167](https://image.imxyu.cn/file/image-20220313185305167.png)

回到我们的源码中，可以看到这里首先获取到了acceptableTypes 就是浏览器中能接收的类型

然后又得到了服务器能产生的类型（这里因为我们的返回值是Person， 所以会看服务器能将这个Person类型转换成什么格式的数据），

然后for循环开始协商。看哪种类型能产生并且浏览器能接收

![image-20220313185152193](https://image.imxyu.cn/file/image-20220313185152193.png)



![image-20220313185716710](https://image.imxyu.cn/file/image-20220313185716710.png)

![image-20220313185824817](https://image.imxyu.cn/file/image-20220313185824817.png)

这里匹配得到了json这种格式的

![image-20220313194027936](https://image.imxyu.cn/file/image-20220313194027936.png)

然后会进行for循环所有的messageConverters，看哪个消息转换器能转换成这种类型的

![image-20220313191338268](https://image.imxyu.cn/file/image-20220313191338268.png)

HttpMessageConverter 

canwrite有两个参数: canWrite看是否支持将 此 Class类型的对象，转为MediaType类型的数据 ，canRead两个 参数看是否能将MediaType类型读出Class类型

然后提供了read 和write的对应方法



例子：Person对象转为JSON。或者 JSON转为Person

![image.png](https://image.imxyu.cn/file/1605163447900-e2748217-0f31-4abb-9cce-546b4d790d0b.png)

默认的messageConverters一共有这么多种，看哪一种能写这种类型的

![image.png](https://image.imxyu.cn/file/1605163584708-e19770d6-6b35-4caa-bf21-266b73cb1ef1.png)

0 - 只支持Byte类型的

1 - String

2 - String

3 - Resource

4 - ResourceRegion

5 - DOMSource.**class \** SAXSource.**class**) \ StAXSource.**class \**StreamSource.**class \**Source.**class**

**6 -** MultiValueMap

7 - **返回true** （都支持）

**8 - 返回true**（都支持）

**9 - 支持注解方式xml处理的。**

经过所有的消息转换器的canWrite 的判断后

最终 MappingJackson2HttpMessageConverter调用它的write方法  把对象转为上面得到的json类型的（利用底层的jackson的objectMapper转换的)

![image-20220313192416273](https://image.imxyu.cn/file/image-20220313192416273.png)



![image-20220313192803795](https://image.imxyu.cn/file/image-20220313192803795.png)



总结：

- 1、返回值处理器判断是否支持这种类型返回值 supportsReturnType
- 2、返回值处理器调用 handleReturnValue 进行处理
- 3、**RequestResponseBodyMethodProcessor** 可以处理返回值标了@ResponseBody 注解的。

- - 1. 利用 MessageConverters 进行处理 将数据写为json

- - - 1、内容协商（浏览器默认会以请求头的方式告诉服务器他能接受什么样的内容类型）
    - 2、服务器最终根据自己自身的能力，决定服务器能生产出什么样内容类型的数据，
    - 3、SpringMVC会挨个遍历所有容器底层的 HttpMessageConverter ，看谁能处理？

- - - - 1、得到MappingJackson2HttpMessageConverter可以将对象写为json
      - 2、利用MappingJackson2HttpMessageConverter将对象转为json再写出去

> 只要我们使用@Responsebody 注解标注的，都会寻找相应的消息转换器进行处理

### @ResponseBody返回值举例：

例如上面我们看到第三个消息转换器可以处理Resource 的返回值，那么我们就可以返回Resource

![image-20220313193459153](https://image.imxyu.cn/file/image-20220313193459153.png)

```java
    @ResponseBody //--RequestResponseBodyMethodProcessor ---> messageConverter--->ResourceHttpMessageConverter帮我们处理
    @GetMapping("/he11")
    public FileSystemResource file(){

        //文件以这样的方式返回看是谁处理的（messageConverter）。
        return null;
    }

```



## 内容协商

根据客户端接收能力不同，返回不同媒体类型的数据。

上面我们讲了当我们引入web-starter 后默认会给我们引入json 的jar包，支持处理json 格式的数据

那么如果我们现在请求端（浏览器、安卓、客户端）想要返回其他类型的数据怎么办呢？ 

例如返回xml 数据

### 1、引入xml依赖

首先我们要引入能处理xml的依赖，引入之后服务端就具备了能将返回类型转换成xml类型的能力

```xml
 <dependency>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

### 2、postman分别测试返回json和xml

只需要改变请求头中Accept字段。Http协议中规定的，告诉服务器本客户端可以接收的数据类型。

![image-20220313201107368](https://image.imxyu.cn/file/image-20220313201107368.png)



可以看到我们通过修改Accept 告诉我们想要接收什么样的头，此时服务端就会帮我们返回相应格式的文件

![image-20220313201126106](https://image.imxyu.cn/file/image-20220313201126106.png)



### 3、【源码】内容协商原理

- 1、判断当前响应头中是否已经有确定的媒体类型。MediaType
- **2、获取客户端（PostMan、浏览器）支持接收的内容类型。（获取客户端Accept请求头字段）【application/xml】**

- - **contentNegotiationManager 内容协商管理器 默认使用基于请求头的策略**
  - ![img](https://image.imxyu.cn/file/1605230462280-ef98de47-6717-4e27-b4ec-3eb0690b55d0.png)
  - **HeaderContentNegotiationStrategy  确定客户端可以接收的内容类型** 
  - ![img](https://image.imxyu.cn/file/1605230546376-65dcf657-7653-4a58-837a-f5657778201a.png)

- 3、遍历循环所有当前系统的 **MessageConverter**，通过调用canWrite方法看谁支持操作这个对象（Person）
- 4、找到支持操作Person的converter，把converter支持的媒体类型统计出来。
- 5、客户端需要【application/xml】。服务端能力【10种、json、xml】
-   ![img](https://image.imxyu.cn/file/1605173876646-f63575e2-50c8-44d5-9603-c2d11a78adae.png)
- 6、双层for循环（根据服务端、客户端两者的类型）进行内容协商的最佳匹配媒体类型
- 7、用 支持 将对象转为 最佳匹配媒体类型 的converter。调用它进行转化 。

第十个就是我们引入XML的依赖之后出现的能转换xml类型的converter



![img](https://image.imxyu.cn/file/1605173657818-73331882-6086-490c-973b-af46ccf07b32.png)

> 具体的源码上面已经分析过了，这里就不一 一截图了

此时我们的浏览器发送请求也会帮我们转成xml 格式的，

因为这里accpet中接收xml 格式占了0.9 也就是说优先让服务器返回这个xml 格式的，如果服务器不能产生这种xml的话，后面还有一个`*/*`表示它可以接收所有的，此时才会转成json返回给它， 

> 这是服务器和客户端两者共同协商决定的，当然如果没有一种能满足的话，就会直接返回错误了

![image-20220313201909497](https://image.imxyu.cn/file/image-20220313201909497.png)

这也解释了为什么我们js  中ajax发请求的时候有时候会带content-type = application/json ， 意思就是希望服务器给它返回json 数据

### 4、开启浏览器参数方式内容协商功能

对于postman 我们很容器模拟出请求头为json或者xml 格式的，但是对于浏览器如果我们也想指定的话怎么办？

为了方便内容协商，开启基于请求参数的内容协商功能。

首先在配置文件中开启内容协商模式

```xml
spring:
  mvc:
    contentnegotiation:
      favor-parameter: true  #开启请求参数内容协商模式
```

发请求： http://localhost:8080/test/person?format=json

[		http://localhost:8080/test/person?format=xml](http://localhost:8080/test/person?format=xml)

我们看一下这里获取accept请求头

![image-20220313203243326](https://image.imxyu.cn/file/image-20220313203243326.png)



![image-20220313203314065](https://image.imxyu.cn/file/image-20220313203314065.png)

这里会有多了一种策略基于参数的请求策略，我们看一下

![image-20220313203430789](https://image.imxyu.cn/file/image-20220313203430789.png)

这里会找到参数中的format 对应的值，这里我们传的是json，此时就会得到json

![image-20220313203818033](https://image.imxyu.cn/file/image-20220313203818033.png)

然后这里会判断如果得到的不是/** 就返回，因此从第一种策略参数策略中就已经得到了客户端需要的值。

因此就不会走第二个获取accpet的策略

![image-20220313203906168](https://image.imxyu.cn/file/image-20220313203906168.png)

下面的就是和服务端的10种转换的converter进行协商的步骤了，就不概述了

> 总结：通过在配置文件中开启这个属性之后，会帮我们添加一个这个参数的策略，通过这个参数的策略我们可以在浏览器的参数上获得format对应的值，从而返回给浏览器需要的格式

并且我们可以看到虽然我们使用了参数上format的方式，但是浏览器发送的请求中accept还是没有改变，还是xml 占0.9，就是因为参数的策略先执行了得到了浏览器的format，后面的请求头的策略就不会执行了

![image-20220313204412694](https://image.imxyu.cn/file/image-20220313204412694.png)



### 5、自定义messageConverter

现在我们有这样下面这种需求根据不同的请求返回不同格式的数据，我们可以写3个方法，让不同的设备调用不同的方法，当然这样比较麻烦。

现在我们可以利用converter 进行自动的转换

```java
   /**
     * 1、浏览器发请求直接返回 xml    [application/xml]        jacksonXmlConverter
     * 2、如果是ajax请求 返回 json   [application/json]      jacksonJsonConverter
     * 3、如果硅谷app发请求，返回自定义协议数据  [appliaction/x-guigu]   xxxxConverter
     *      返回这种格式的：    属性值1;属性值2;
     *	
     * 想一下前两种springboot底层的converter就可以满足要求了，根据请求头中accept中的不同可以返回对应的格式，而对于第三种我们就需要自定义一个converter进行转换
     	
     步骤：
     * 1、添加自定义的MessageConverter进系统底层
     * 2、系统底层就会统计出所有MessageConverter能操作哪些类型
     * 3、客户端内容协商 [guigu--->guigu]
```

此时我们就要自定义一个converter ，去处理上面的这种guigu 的请求[appliaction/x-guigu] 这种格式的请求头

首先我们来看一下底层的这些converter是怎么注入的。

#### 默认的converter注入原理

在自动配置类中直接添加了所有的converter， 我们点进去这个getConverter方法

![image-20220313205942800](https://image.imxyu.cn/file/image-20220313205942800.png)

![image-20220313210412808](https://image.imxyu.cn/file/image-20220313210412808.png)

而这个converters 是这个对象创建的时候获取默认的converter

![image-20220313210432988](https://image.imxyu.cn/file/image-20220313210432988.png)



一直往里点，最后发现这个方法，可以看到这里面add了很多converter 

并且还有根据条件判断的添加converter

![image-20220313210612230](https://image.imxyu.cn/file/image-20220313210612230.png)

可以看到这个条件判断就是判断是否存在这个包，也就是说如果我们引入这个包了就会帮我们添加converter。

这也就解释了上面为什么我们添加了一个能处理xml 的pom 文件就能帮我们创建这个converter

![image-20220313210715273](https://image.imxyu.cn/file/image-20220313210715273.png)

#### 自定义converter

在webmvc自动配置类中，我们可以使用这两个方法

第一个方法会直接覆盖所有的消息转换器，那么我们只希望添加一个我们的（扩展），所以我们可以使用第二个extendMessageConverters

![image-20220313211019506](https://image.imxyu.cn/file/image-20220313211019506.png)

自定义webmvc的组件，我们还是使用之前的方法，，此时我们需要创建一个自定义的messageConverter 注册进去

![image-20220313211215207](https://image.imxyu.cn/file/image-20220313211215207.png)



创建自定义的converter

```java
/**
 * 自定义的Converter
 */
public class GuiguMessageConverter implements HttpMessageConverter<Person> {

    @Override
    public boolean canRead(Class<?> clazz, MediaType mediaType) {
        return false;
    }

    @Override
    public boolean canWrite(Class<?> clazz, MediaType mediaType) {
        //表明所有返回的class类型为Person的，我们都可以写
       
        return clazz.isAssignableFrom(Person.class);
    }

    /**
     * 服务器要统计所有MessageConverter都能写出哪些内容类型
     *
     * application/x-guigu 我们支持这种的mediaType
     * @return
     */
    @Override
    public List<MediaType> getSupportedMediaTypes() {
        return MediaType.parseMediaTypes("application/x-guigu");
    }

    @Override
    public Person read(Class<? extends Person> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
        return null;
    }

    @Override
    public void write(Person person, MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
        //自定义协议数据的写出
        String data = person.getUserName()+";"+person.getAge()+";"+person.getBirth();

        //写出去
        OutputStream body = outputMessage.getBody();
        body.write(data.getBytes());
    }
}

```

> 什么时候会用到读呢？ 比如我们在参数上写的@RequestBody 注解的话，会调用所有的converter的canRead方法 ，看哪一个能将mediaType 类型转换成对应的class类型。 我们之前经常会使用@RequestBody 将post请求体中的数据转换成我们想要的数据就是利用这个
>
> 此时我们不想演示读的操作，我们只重写了write 的操作

这里使用了我们第二种策略，因为我们没有传递format 参数，所以会使用第二个header策略获取到accept中的值

![image-20220313212537895](https://image.imxyu.cn/file/image-20220313212537895.png)

![image-20220313212558398](https://image.imxyu.cn/file/image-20220313212558398.png)

此时会经过下面两层for循环后，确定了mediaType类型为x-guigu 

![image-20220313212624227](https://image.imxyu.cn/file/image-20220313212624227.png)

通过CanWrite 进行判断，最终发现我们的converter能处理，最后调用我们的处理方法

![image-20220313212655412](https://image.imxyu.cn/file/image-20220313212655412.png)

可以看到这就是返回的我们处理后通过分号拼接的字符串格式

![image-20220313212810142](https://image.imxyu.cn/file/image-20220313212810142.png)



#### 自定义converter兼容浏览器

我们可以看到第一种参数的策略只支持这两种类型

![image-20220313213807686](https://image.imxyu.cn/file/image-20220313213807686.png)

可以看到如果我们在fomat 中随意传一个类型比如xx ，会在这个支持的mediaTypes 中找，因为只支持那两种类型，所以找不到

![image-20220313214040131](https://image.imxyu.cn/file/image-20220313214040131.png)

那么如何兼容呢？让浏览器携带format 也能返回我们自定义的格式呢

因为原参数解析的策略中只支持两个，因此我们直接覆盖这个ContentNegotiation这个获取的组件，然后我们自己创建一个ParameterContentNegotiationStrategy并且添加能支持的mediaType 就可以了

```java
 @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {

            /**
             * 自定义内容协商策略
             * @param configurer
             */
            @Override
            public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
                //Map<String, MediaType> mediaTypes
                Map<String, MediaType> mediaTypes = new HashMap<>();
                mediaTypes.put("json",MediaType.APPLICATION_JSON);
                mediaTypes.put("xml",MediaType.APPLICATION_XML);
                mediaTypes.put("gg",MediaType.parseMediaType("application/x-guigu"));
                //指定支持解析哪些参数对应的哪些媒体类型
                ParameterContentNegotiationStrategy parameterStrategy = new ParameterContentNegotiationStrategy(mediaTypes);
//                parameterStrategy.setParameterName("ff");
				//这里还需要把第二个策略根据头的添加上，否则的话如果我们使用post-man 不携带参数发送的话经过第一个策略会返回*/*,会自动匹配第一条converter转换器，不管我们传什么类型的accept 头都会给我们json数据
                HeaderContentNegotiationStrategy headeStrategy = new HeaderContentNegotiationStrategy();

                configurer.strategies(Arrays.asList(parameterStrategy,headeStrategy));
            }

            @Override
            public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
                converters.add(new GuiguMessageConverter());
            }
.....
        }
```

此时我们传入gg， 就会转换成application/x-guigu 类型

![image-20220313215756670](https://image.imxyu.cn/file/image-20220313215756670.png)

**有可能我们添加的自定义的功能会覆盖默认很多功能，导致一些默认的功能失效。**

**大家考虑，上述功能除了我们完全自定义外？SpringBoot有没有为我们提供基于配置文件的快速修改媒体类型功能？怎么配置呢？**

我们可以在配置文件中使用这个配置项进行添加

```
spring.mvc.contentnegotiation.media-types.gg=application/x-guigu
```

总结：

**实现多协议数据兼容。json、xml、x-guigu**

0、@ResponseBody 响应数据出去 调用 **RequestResponseBodyMethodProcessor** 处理

1、Processor 处理方法返回值。通过 **MessageConverter** 处理

2、所有 **MessageConverter** 合起来可以支持各种媒体类型数据的操作（读、写）

3、内容协商找到最终的 **messageConverter**；



# 视图解析与模板引擎

springboot 打出来的是jar包， 而jsp 不支持在jar包中编译的方式

视图解析：**SpringBoot默认不支持 JSP，需要引入第三方模板引擎技术实现页面渲染。**

## 简介

Thymeleaf is a modern server-side Java template engine for both web and standalone environments, capable of processing HTML, XML, JavaScript, CSS and even plain text.

**现代化、服务端Java模板引擎**

> 优点是非常接近jsp 的语法，一般后端会jsp的话，th的语法也很容易上手
>
> 缺点是性能比较差，如果做分布式高并发的项目的话还是需要前后端分离

## 语法

### 基本语法

#### 1、表达式

| 表达式名字 | 语法   | 用途                               |
| ---------- | ------ | ---------------------------------- |
| 变量取值   | ${...} | 获取请求域、session域、对象等值    |
| 选择变量   | *{...} | 获取上下文对象值                   |
| 消息       | #{...} | 获取国际化等值                     |
| 链接       | @{...} | 生成链接                           |
| 片段表达式 | ~{...} | jsp:include 作用，引入公共页面片段 |

#### 2、字面量

文本值: **'one text'** **,** **'Another one!'** **,…**数字: **0** **,** **34** **,** **3.0** **,** **12.3** **,…**布尔值: **true** **,** **false**

空值: **null**

变量： one，two，.... 变量不能有空格

#### 3、文本操作

字符串拼接: **+**

变量替换: **|The name is ${name}|** 



#### 4、数学运算

运算符: + , - , * , / , %



#### 5、布尔运算

运算符:  **and** **,** **or**

一元运算: **!** **,** **not** 



#### 6、比较运算

比较: **>** **,** **<** **,** **>=** **,** **<=** **(** **gt** **,** **lt** **,** **ge** **,** **le** **)**等式: **==** **,** **!=** **(** **eq** **,** **ne** **)** 



#### 7、条件运算

If-then: **(if) ? (then)**

If-then-else: **(if) ? (then) : (else)**

Default: (value) **?: (defaultvalue)** 



#### 8、特殊操作

无操作： _



#### 行内写法

<p>Hello, [[${session.user.name}]]!</p>



### 设置属性值-th:attr

**设置单个值**

```html
<form action="subscribe.html" th:attr="action=@{/subscribe}">
  <fieldset>
    <input type="text" name="email" />
    <input type="submit" value="Subscribe!" th:attr="value=#{subscribe.submit}"/>
  </fieldset>
</form>
```

**设置多个值**

```html
<img src="../../images/gtvglogo.png"  th:attr="src=@{/images/gtvglogo.png},title=#{logo},alt=#{logo}" />
```

**以上两个的代替写法 th:xxxx (推荐)**

```html
<input type="submit" value="Subscribe!" th:value="#{subscribe.submit}"/>
<form action="subscribe.html" th:action="@{/subscribe}">
```

所有h5兼容的标签写法

https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#setting-value-to-specific-attributes



### 迭代

```html
<tr th:each="prod : ${prods}">
        <td th:text="${prod.name}">Onions</td>
        <td th:text="${prod.price}">2.41</td>
        <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>
```



```html
<tr th:each="prod,iterStat : ${prods}" th:class="${iterStat.odd}? 'odd'">
  <td th:text="${prod.name}">Onions</td>
  <td th:text="${prod.price}">2.41</td>
  <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>
```



### 条件运算

```html
<a href="comments.html"
th:href="@{/product/comments(prodId=${prod.id})}"
th:if="${not #lists.isEmpty(prod.comments)}">view</a>
```



```html
<div th:switch="${user.role}">
  <p th:case="'admin'">User is an administrator</p>
  <p th:case="#{roles.manager}">User is a manager</p>
  <p th:case="*">User is some other thing</p>
</div>
```



### 属性优先级

![image.png](https://image.imxyu.cn/file/1605498132699-4fae6085-a207-456c-89fa-e571ff1663da.png)

## thymeleaf使用

###  引入starter

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

thymeleaf自动配置类自动配置好了thymeleaf

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(ThymeleafProperties.class)
@ConditionalOnClass({ TemplateMode.class, SpringTemplateEngine.class })
@AutoConfigureAfter({ WebMvcAutoConfiguration.class, WebFluxAutoConfiguration.class })
public class ThymeleafAutoConfiguration { }
```

自动配好的策略

- 1、所有thymeleaf的配置值都在 ThymeleafProperties
- 2、配置好了 **SpringTemplateEngine** 
- **3、配好了** **ThymeleafViewResolver** 
- 4、我们只需要直接开发页面

可以看到绑定的properties 中指定好了文件的默认路径和后缀

```java
	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";  //xxx.html
```

### 页面开发初体验

在temlates/ 文件夹下创建html页面，并且添加名称空间 头文件xmlns:th="http://www.thymeleaf.org" 之后就有了th的提示

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 th:text="${msg}">哈哈</h1>
<h2>
    <a href="www.atguigu.com" th:href="${link}">去百度</a>  <br/>
    <a href="www.atguigu.com" th:href="@{link}">去百度2</a>
</h2>
</body>
</html>
```

测试通过后台向前台页面中传数据

```java
@Controller
public class ViewTestController {

    @GetMapping("/atguigu")
    public String atguigu(Model model){

        //model中的数据会被放在请求域中 request.setAttribute("a",aa)
        model.addAttribute("msg","你好 guigu");
        model.addAttribute("link","http://www.baidu.com");
        return "success";
    }
}
```

其中@{link} 可以生成连接，{ }括号中我们可以写相应的链接就可以，这里如果我们写 / +链接地址，表示的是相对连接，相对当前浏览器的地址 

可以看到经过html 页面渲染后取到了后台的数据渲染后的结果

![image-20220314112154586](https://image.imxyu.cn/file/image-20220314112154586.png)

### 构建后台管理系统

#### 项目创建

使用IDEA的Spring Initializr。

- thymeleaf、
- web-starter、
- devtools、
- lombok

#### 登陆页面

- `/static` 放置 css，js等静态资源
- `/templates/login.html` 登录页

```html

<html lang="en" xmlns:th="http://www.thymeleaf.org"><!-- 要加这玩意thymeleaf才能用 -->

<form class="form-signin" action="index.html" method="post" th:action="@{/login}">

    ...
    
    <!-- 消息提醒 -->
    <label style="color: red" th:text="${msg}"></label>
    
    <input type="text" name="userName" class="form-control" placeholder="User ID" autofocus>
    <input type="password" name="password" class="form-control" placeholder="Password">
    
    <button class="btn btn-lg btn-login btn-block" type="submit">
        <i class="fa fa-check"></i>
    </button>
    
    ...
    
</form>


```

- `/templates/main.html` 主页

thymeleaf内联写法：

```html
<p>Hello, [[${session.user.name}]]!</p>
```

#### 登录控制层

这里需要注意的是，当我们登录成功之后为了防止表单的重复提交，我们需要重定向到主页，而不是转发到主页。

因为转发的话页面地址不变，页面地址还是login并且转发request 中的值也会携带，重新刷新页面会又进行一次登录。

解决重复提交的最好方法就是使用重定向，而我们的thymleaf 会给我们拼接头和后缀，我们不能直接到main.html 页面，所以我们需要再写一个方法利用这个方法转发过去

```java

@Controller
public class IndexController {
    /**
     * 来登录页
     * @return
     */
    @GetMapping(value = {"/","/login"})
    public String loginPage(){

        return "login";
    }

    @PostMapping("/login")
    public String main(User user, HttpSession session, Model model){ //RedirectAttributes
		
        if(StringUtils.hasLength(user.getUserName()) && "123456".equals(user.getPassword())){
            //把登陆成功的用户保存起来
            //同时会生成一个sessionid, 返回给浏览器。下次浏览器都会带着这个sessionid来找服务端。服务端根据sessionId可以找到对应session中的属性
            session.setAttribute("loginUser",user);
            //登录成功重定向到main.html;  重定向防止表单重复提交
            return "redirect:/main.html";
        }else {
            model.addAttribute("msg","账号密码错误");
            //回到登录页面
            return "login";
        }
    }
    
     /**
     * 去main页面
     * @return
     */
    @GetMapping("/main.html")
    public String mainPage(HttpSession session, Model model){
        
        //最好用拦截器,过滤器
        //下次请求来了之后会根据浏览器携带的sessionId找到对应的session，取出session中设置的属性
        Object loginUser = session.getAttribute("loginUser");
        if(loginUser != null){
        	return "main";
        }else {
            //session过期，没有登陆过
        	//回到登录页面
	        model.addAttribute("msg","请重新登录");
    	    return "login";
        }
    }
    
}


```

> 使用session ，可以让用户下次访问不再需要输入用户名密码的情况

#### pojo

```java

@Data
public class User {
    private String userName;
    private String password;
}
```



#### 抽取公共页面

把公共的css, js ， 左边的菜单栏和顶部导航栏都抽取到这一个页面中。后面使用的时候直接引入即可

修改的话只需要修改这一个页面即可。**两种方式抽取页面**

1、th:fragment="headermenu"      th:include="common :: commonheader  

2、直接定义id   th:replace="common :: #commonscript"  使用#取id的

```html

<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org"><!--注意要添加xmlns:th才能添加thymeleaf的标签-->
<head th:fragment="commonheader">
    <!--common-->
    <link href="css/style.css" th:href="@{/css/style.css}" rel="stylesheet">
    <link href="css/style-responsive.css" th:href="@{/css/style-responsive.css}" rel="stylesheet">
    ...
</head>
<body>
<!-- left side start-->
<div id="leftmenu" class="left-side sticky-left-side">
	...

    <div class="left-side-inner">
		...

        <!--sidebar nav start-->
        <ul class="nav nav-pills nav-stacked custom-nav">
            <li><a th:href="@{/main.html}"><i class="fa fa-home"></i> <span>Dashboard</span></a></li>
            ...
            <li class="menu-list nav-active"><a href="#"><i class="fa fa-th-list"></i> <span>Data Tables</span></a>
                <ul class="sub-menu-list">
                    <li><a th:href="@{/basic_table}"> Basic Table</a></li>
                    <li><a th:href="@{/dynamic_table}"> Advanced Table</a></li>
                    <li><a th:href="@{/responsive_table}"> Responsive Table</a></li>
                    <li><a th:href="@{/editable_table}"> Edit Table</a></li>
                </ul>
            </li>
            ...
        </ul>
        <!--sidebar nav end-->
    </div>
</div>
<!-- left side end-->


<!-- header section start-->
<div th:fragment="headermenu" class="header-section">

    <!--toggle button start-->
    <a class="toggle-btn"><i class="fa fa-bars"></i></a>
    <!--toggle button end-->
	...

</div>
<!-- header section end-->

<div id="commonscript">
    <!-- Placed js at the end of the document so the pages load faster -->
    <script th:src="@{/js/jquery-1.10.2.min.js}"></script>
    <script th:src="@{/js/jquery-ui-1.9.2.custom.min.js}"></script>
    <script th:src="@{/js/jquery-migrate-1.2.1.min.js}"></script>
    <script th:src="@{/js/bootstrap.min.js}"></script>
    <script th:src="@{/js/modernizr.min.js}"></script>
    <script th:src="@{/js/jquery.nicescroll.js}"></script>
    <!--common scripts for all pages-->
    <script th:src="@{/js/scripts.js}"></script>
</div>
</body>
</html>


```

- `/templates/table/basic_table.html`

```html

<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
  <meta name="description" content="">
  <meta name="author" content="ThemeBucket">
  <link rel="shortcut icon" href="#" type="image/png">

  <title>Basic Table</title>
    <div th:include="common :: commonheader"> </div><!--将common.html的代码段 插进来-->
</head>

<body class="sticky-header">

<section>
<div th:replace="common :: #leftmenu"></div>
    
    <!-- main content start-->
    <div class="main-content" >

        <div th:replace="common :: headermenu"></div>
        ...
    </div>
    <!-- main content end-->
</section>

<!-- Placed js at the end of the document so the pages load faster -->
<div th:replace="common :: #commonscript"></div>


</body>
</html>



```

可以使用这三种方式引入

[Difference between `th:insert` and `th:replace` (and `th:include`)](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#difference-between-thinsert-and-threplace-and-thinclude)



#### 遍历数据

```java

@GetMapping("/dynamic_table")
public String dynamic_table(Model model){
    //表格内容的遍历
    List<User> users = Arrays.asList(new User("zhangsan", "123456"),
                                     new User("lisi", "123444"),
                                     new User("haha", "aaaaa"),
                                     new User("hehe ", "aaddd"));
    model.addAttribute("users",users);

    return "table/dynamic_table";
}


```

页面代码

th:each="user,stats:${users}"  stats 表示循环的其他数据

```html


<table class="display table table-bordered" id="hidden-table-info">
    <thead>
        <tr>
            <th>#</th>
            <th>用户名</th>
            <th>密码</th>
        </tr>
    </thead>
    <tbody>
        <tr class="gradeX" th:each="user,stats:${users}">
            <td th:text="${stats.count}">Trident</td>
            <td th:text="${user.userName}">Internet</td>
            <td >[[${user.password}]]</td>
        </tr>
    </tbody>
</table>


```

## 视图解析原理【源码】

上面的这个处理请求中的参数赋值，然后调用方法我们就不看了。我们从这里处理返回值来看

![image-20220314144345001](https://image.imxyu.cn/file/image-20220314144345001.png)

这里会将通过反射调用方法后的返回值拿到，然后在returnValueHandlers 中 看哪个能处理这个返回值，最终找到了这个ViewNameMethodReturnValueHandler

![image-20220314144453947](https://image.imxyu.cn/file/image-20220314144453947.png)

我们看一下这个返回值处理器能处理什么类型的，可以看到它支持处理void 和字符串类型

并且找到后调用处理的逻辑，我们也能看到就是将这个字符串设置到这个mavContainer

![image-20220314144627880](https://image.imxyu.cn/file/image-20220314144627880.png)



上面的invokeAndHandle之后就会给view 中放入这个viewName 。我们继续向下看返回modelAndView 的流程

![image-20220314144903122](https://image.imxyu.cn/file/image-20220314144903122.png)

这里会将mavContainer 中的model 和view 都拿出来，封装成一个modelAndView 对象，返回

![image-20220314145017264](https://image.imxyu.cn/file/image-20220314145017264.png)

然后调用下面的这个处理modelAndView 对象返回视图的流程

![image-20220314145150127](https://image.imxyu.cn/file/image-20220314145150127.png)

渲染视图

![image-20220314145219443](https://image.imxyu.cn/file/image-20220314145219443.png)

根据视图名找到能处理的view对象，看看是如何找的

![image-20220314145249215](https://image.imxyu.cn/file/image-20220314145249215.png)

拿到了所有的视图解析器，判断是否能处理这个view Name,如果能处理的话就返回得到一个View 对象

![image-20220314145326933](https://image.imxyu.cn/file/image-20220314145326933.png)

可以看到这里根绝ThymeleafViewResolver返回了RedirectView 对象

![image-20220314145428515](https://image.imxyu.cn/file/image-20220314145428515.png)

调用跟这个view的render方法渲染

![image-20220314145704334](https://image.imxyu.cn/file/image-20220314145704334.png)



![image-20220314145738539](https://image.imxyu.cn/file/image-20220314145738539.png)

重定向到main.html页面

![image-20220314145834091](https://image.imxyu.cn/file/image-20220314145834091.png)



1、目标方法处理的过程中，所有数据都会被放在 **ModelAndViewContainer 里面。包括数据和视图地址**

**2、方法的参数是一个自定义类型对象（从请求参数中确定的），把他重新放在** **ModelAndViewContainer** 

**3、任何目标方法执行完成以后都会返回 ModelAndView（数据和视图地址）。**

**4、processDispatchResult  处理派发结果（页面改如何响应）**

- 1、**render**(**mv**, request, response); 进行页面渲染逻辑

- - 1、根据方法的String返回值得到 **View** 对象【定义了页面的渲染逻辑】

- - - 1、所有的视图解析器尝试是否能根据当前返回值得到**View**对象
    - 2、得到了  **redirect:/main.html** --> Thymeleaf new **RedirectView**()
    - 3、ContentNegotiationViewResolver 里面包含了下面所有的视图解析器，内部还是利用下面所有视图解析器得到视图对象。
    - 4、view.render(mv.getModelInternal(), request, response);   视图对象调用自定义的render进行页面渲染工作

- - - - **RedirectView 如何渲染【重定向到一个页面】**
      - **1、获取目标url地址**
      - **2、response.sendRedirect(encodedURL);**





**thymeleaf视图解析器的resolve判断：**

- - **返回值以 forward: 开始： new InternalResourceView(forwardUrl); -->  转发**request.getRequestDispatcher(path).forward(request, response);
  - **返回值以** **redirect: 开始：** **new RedirectView() --》 render就是重定向** 
  - **返回值是普通字符串： new ThymeleafView（）--->**  将数据write出去，变成html页面

自定义视图：看大厂学院

## 拦截器

对于我们的所有页面，都要进行登录以后才能访问，目前我们只在主页面进行了拦截，如果要对所有的请求都写的话。比较麻烦，我们可以使用拦截器的功能

```java

@Controller
public class IndexController {
    /**
     * 来登录页
     * @return
     */
    @GetMapping(value = {"/","/login"})
    public String loginPage(){

        return "login";
    }

    @PostMapping("/login")
    public String main(User user, HttpSession session, Model model){ //RedirectAttributes
		
        if(StringUtils.hasLength(user.getUserName()) && "123456".equals(user.getPassword())){
            //把登陆成功的用户保存起来
            //同时会生成一个sessionid, 返回给浏览器。下次浏览器都会带着这个sessionid来找服务端。服务端根据sessionId可以找到对应session中的属性
            session.setAttribute("loginUser",user);
            //登录成功重定向到main.html;  重定向防止表单重复提交
            return "redirect:/main.html";
        }else {
            model.addAttribute("msg","账号密码错误");
            //回到登录页面
            return "login";
        }
    }
    
     /**
     * 去main页面
     * @return
     */
    @GetMapping("/main.html")
    public String mainPage(HttpSession session, Model model){
        
        //最好用拦截器,过滤器
        //下次请求来了之后会根据浏览器携带的sessionId找到对应的session，取出session中设置的属性
        Object loginUser = session.getAttribute("loginUser");
        if(loginUser != null){
        	return "main";
        }else {
            //session过期，没有登陆过
        	//回到登录页面
	        model.addAttribute("msg","请重新登录");
    	    return "login";
        }
    }
    
}

```

### 1、实现HandlerInterceptor 接口

这个接口就是我们的拦截器接口

拦截器的执行顺序先执行1、preHandle()   2、目标方法  3、postHandle（） 4、页面调用render渲染  5、渲染后调用这个afterCompletion（）

因此我们的登录逻辑需要在preHandle（）中写

```java
/**
 * 登录检查
 * 1、配置好拦截器要拦截哪些请求
 * 2、把这些配置放在容器中
 */
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {

    /**
     * 目标方法执行之前
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        log.info("preHandle拦截的请求路径是{}",requestURI);

        //登录检查逻辑
        HttpSession session = request.getSession();

        Object loginUser = session.getAttribute("loginUser");

        if(loginUser != null){
            //放行
            return true;
        }

        //拦截住。未登录。跳转到登录页
        request.setAttribute("msg","请先登录");
//        re.sendRedirect("/");
        request.getRequestDispatcher("/").forward(request,response);
        return false;
    }

    /**
     * 目标方法执行完成以后
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle执行{}",modelAndView);
    }

    /**
     * 页面渲染以后
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("afterCompletion执行异常{}",ex);
    }
}
```

### 2、配置拦截器

同样我们自定义组件可以在配置类中实现这个接口、或者@Bean创建WebMvcConfigurer组件 都行，这里我们使用实现接口。

里面有一个addInterceptors方法，添加我们的自定义拦截器，在链式请求返回的时候添加要拦截的路径（这里使用/** 拦截了所有），

然后要排除我们的静态资源路径和登录页面的两个请求路径

```java
/**
 * 1、编写一个拦截器实现HandlerInterceptor接口
 * 2、拦截器注册到容器中（实现WebMvcConfigurer的addInterceptors）
 * 3、指定拦截规则【如果是拦截所有，静态资源也会被拦截】
 */
@Configuration
public class AdminWebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")  //所有请求都被拦截包括静态资源
                .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**","/js/**"); //放行的请求
    }
}
```

静态资源的放行我们也可以使用在配置文件中定义这个前缀的方式来实现，如果定义了前缀的话，我们的页面上的所有资源前面的路径都要加上这个/static

![image-20220314152659939](https://image.imxyu.cn/file/image-20220314152659939.png)

需要修改所有的对于静态资源的引用页面加上/static

![image-20220314153147157](https://image.imxyu.cn/file/image-20220314153147157.png)

### 3、拦截器原理【源码】

当我们发送一个请求时，可以看到在getHandler 中不仅拿到了我们的映射处理器，还拿到了我们的拦截器链

![image-20220314154726759](https://image.imxyu.cn/file/image-20220314154726759.png)



可以看到在方法的执行前后会执行我们的拦截器的方法

![image-20220314155044159](https://image.imxyu.cn/file/image-20220314155044159.png)

for 循环按照顺序执行拦截器链， 如果有一处出现问题，会记录当前出现问题的索引，然后调用下面的triggerAfterCompletion方法

![image-20220314155110439](https://image.imxyu.cn/file/image-20220314155110439.png)

这个方法会反向调用从出现问题的地方之前 的所有拦截器的triggerAfterCompletion方法

> 也就是说能够放行的拦截器的所有方法都会执行的，不会因为某一个炸了。前面的方法会不执行

![image-20220314155155717](https://image.imxyu.cn/file/image-20220314155155717.png)

渲染视图后会执行所有拦截器的applyAfterConcurrentHandlingStarted方法，倒序执行

![image-20220314155401260](https://image.imxyu.cn/file/image-20220314155401260.png)

1、根据当前请求，找到**HandlerExecutionChain【**可以处理请求的handler以及handler的所有 拦截器】

2、先来**顺序执行** 所有拦截器的 preHandle方法

- 1、如果当前拦截器prehandler返回为true。则执行下一个拦截器的preHandle
- 2、如果当前拦截器返回为false。直接    倒序执行所有已经执行了的拦截器的  afterCompletion；

**3、如果任何一个拦截器返回false。直接跳出不执行目标方法**

**4、所有拦截器都返回True。执行目标方法**

**5、倒序执行所有拦截器的postHandle方法。**

**6、前面的步骤有任何异常都会直接倒序触发** afterCompletion

7、页面成功渲染完成以后，也会倒序触发 afterCompletion



![img](https://image.imxyu.cn/file/1605764129365-5b31a748-1541-4bee-9692-1917b3364bc6.png)

**流程图**

![img](https://image.imxyu.cn/file/1605765121071-64cfc649-4892-49a3-ac08-88b52fb4286f.png)



在执行preHandle 和postHandle 期间有一个return false ，就会倒序执行从当前false 之前的所有拦截器的afterCompletion()

## 文件上传

#### 1、页面表单

这里几个注意点： 1、method 必须为post 2、 enctype="multipart/form-data" 3、如果是多个文件需要指定 <input type="file" name="photos" **multiple**>

```html
 <div class="col-lg-6">
                <section class="panel">
                    <header class="panel-heading">
                        Basic Forms
                    </header>
                    <div class="panel-body">
                        <form role="form" th:action="@{/upload}" method="post" enctype="multipart/form-data">
                            <div class="form-group">
                                <label for="exampleInputEmail1">邮箱</label>
                                <input type="email" name="email" class="form-control" id="exampleInputEmail1" placeholder="Enter email">
                            </div>
                            <div class="form-group">
                                <label for="exampleInputPassword1">名字</label>
                                <input type="text" name="username" class="form-control" id="exampleInputPassword1" placeholder="Password">
                            </div>
                            <div class="form-group">
                                <label for="exampleInputFile">头像</label>
                                <input type="file" name="headerImg" id="exampleInputFile">
                            </div>
                            <div class="form-group">
                                <label for="exampleInputFile">生活照</label>
                                <input type="file" name="photos" multiple>
                            </div>
                            <div class="checkbox">
                                <label>
                                    <input type="checkbox"> Check me out
                                </label>
                            </div>
                            <button type="submit" class="btn btn-primary">提交</button>
                        </form>

                    </div>
                </section>
            </div>
```

#### 2、后台代码

通过@RequestPart 就可以获得文件部分类型，如果是多个文件的话可以使用数组接收

```java
    /**
     * MultipartFile 自动封装上传过来的文件
     * @param email
     * @param username
     * @param headerImg
     * @param photos
     * @return
     */
    @PostMapping("/upload")
    public String upload(@RequestParam("email") String email,
                         @RequestParam("username") String username,
                         @RequestPart("headerImg") MultipartFile headerImg,
                         @RequestPart("photos") MultipartFile[] photos) throws IOException {

        log.info("上传的信息：email={}，username={}，headerImg={}，photos={}",
                email,username,headerImg.getSize(),photos.length);

        if(!headerImg.isEmpty()){
            //保存到文件服务器，OSS服务器
            String originalFilename = headerImg.getOriginalFilename();
            //通过transferTo 方法就可以
            headerImg.transferTo(new File("H:\\cache\\"+originalFilename));
        }

        if(photos.length > 0){
            for (MultipartFile photo : photos) {
                if(!photo.isEmpty()){
                    String originalFilename = photo.getOriginalFilename();
                    photo.transferTo(new File("H:\\cache\\"+originalFilename));
                }
            }
        }


        return "main";
    }

```



#### 3、配置文件中根据需要设置请求、文件大小

我们首先来看一下文件上传自动配置类，绑定的@EnableConfigurationProperties(MultipartProperties.class) 这个文件

![image-20220314161734624](https://image.imxyu.cn/file/image-20220314161734624.png)

所有文件上传功能的配置文件是以这个前缀的，可以看到这里的最大文件大小和最大请求大小分别是1MB，和10MB

![image-20220314161922280](https://image.imxyu.cn/file/image-20220314161922280.png)

在配置文件中自定义参数

![image-20220314162019489](https://image.imxyu.cn/file/image-20220314162019489.png)

#### 4、文件上传原理【源码】

在自动配置类中类中可以看到如果没有这几个类型的组件会自动帮我们放进去

![image-20220314162322882](https://image.imxyu.cn/file/image-20220314162322882.png)

首先当文件上传请求来了之后，会进行判断checkMultipart（request），判断完了之后会将我们的请求封装成processedRequest, 后面会用我们封装的这个processedRequest

![image-20220314163152397](https://image.imxyu.cn/file/image-20220314163152397.png)



![image-20220314163217812](https://image.imxyu.cn/file/image-20220314163217812.png)

判断的流程就是通过判断看request的contetnType 是否是以multipart/开头的，

因此我们的文件上传的表单，必须要使用这个contentType， 就是因为这里

![image-20220314163234488](https://image.imxyu.cn/file/image-20220314163234488.png)

接下来我们看一下这个文件是怎么给我们处理的，我们直接去看参数解析的流程，在24个参数解析器中找到RequestPartMethodArgumentResolver这个参数解析器

![image-20220314163652836](https://image.imxyu.cn/file/image-20220314163652836.png)

调用处理参数的流程

![image-20220314163806876](https://image.imxyu.cn/file/image-20220314163806876.png)

在处理请求的时候会直接根据我们的key调用获取文件的这个map中的value就是封装好的**MultipartFile**，这个map 里面已经封装好了我们的文件根据key ,val

![image-20220314164042036](https://image.imxyu.cn/file/image-20220314164042036.png)

后续我们可以得到文件直接进行如下操作，就是因为multipart接口给我们定义的这些方法

![image-20220314164440820](https://image.imxyu.cn/file/image-20220314164440820.png)

其中transferTo 就是调用spring自己的文件拷贝的工具类FileCopyUtils

> 以后我们有需要也可以使用spring 封装好的工具类进行操作 FileCopyUtils

![image-20220314164512935](https://image.imxyu.cn/file/image-20220314164512935.png)

文件上传自动配置类-**MultipartAutoConfiguration**-**MultipartProperties**

- 自动配置好了 **StandardServletMultipartResolver   【文件上传解析器】**
- **原理步骤**

- - **1、请求进来使用文件上传解析器判断（**isMultipart**）并封装（**resolveMultipart，**返回**MultipartHttpServletRequest**）文件上传请求**
  - **2、参数解析器来解析请求中的文件内容封装成MultipartFile**
  - **3、将request中文件信息封装为一个Map；**MultiValueMap<String, MultipartFile>

**FileCopyUtils**。实现文件流的拷贝

![image.png](https://image.imxyu.cn/file/1605847414866-32b6cc9c-5191-4052-92eb-069d652dfbf9.png)

### 
