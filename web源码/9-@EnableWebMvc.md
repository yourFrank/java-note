上一节中我们实现了自定义了视图解析器，但是我们自定义完之后容器中默认的组件就不创建了，这导致我们失去了很多功能

我们现在的期望：

* 自己的组件能生效
* SpringMvc默认的也能生效

通过@EnableWebMvc注解+实现WebMvcConfigurer 接口即可

我们可以通过过在配置类上标注@EnableWebMvc，并且实现WebMvcConfigurer接口的方法即可

```java
/**
 * 我的视图解析器和SpringMVC默认的都在一起
 *
 * 容器中有 WebMvcConfigurer 类型的组件（@Component\@Configuration）就行
 */

@EnableWebMvc //启用SpringMVC功能，修改SpringMVC底层行为就会很方便只需要实现 WebMvcConfigurer 即可
// 不要这个注解回到以前默认模式，所有组件DispatcherServlet初始化的时候没有，直接用配置文件中指定的默认的组件
// 没有预留扩展接口，需要我们自己重新替换


//1、WebMvcConfigurer+@EnableWebMvc 定制和扩展SpringMVC功能
//2、@EnableWebMvc导入的类会加入SpringMVC的很多核心组件，拥有默认功能
//3、这些默认功能在扩展的时候都是留给接口 WebMvcConfigurer（访问者，拿到真正的内容进行修改） 可以介入
//4、MeiNvViewResolver+InternalResourceViewResolver
//5、@EnableWebMvc开启了SpringMVC === <mvc:annotation-driven/>，即使是以前自己也要配置默认视图解析器
@Configuration 
public class MvcExtendConfiguration implements WebMvcConfigurer {

// 另外一种
//@Configuration 	MvcExtendConfiguration extends DelegatingWebMvcConfiguration
// 1、拿到父类@Bean的方法，还是给容器中放了组件
// 2、只是为了实现一个效果，就是让 DelegatingWebMvcConfiguration 的类或者子类放在容器中，
// 3、只要这个 DelegatingWebMvcConfiguration 生效，他从容器中拿所有的configure进行
// 4、两种方式把 DelegatingWebMvcConfiguration 搞进来 + 一种方式自己写扩展
	//1）、随便在哪个配置类位置加 @EnableWebMvc，然后只需要给 容器中放 WebMvcConfigurer即可
	//2）、自己写一个配置类（在容器中）来继承 DelegatingWebMvcConfiguration，然后只需要给 容器中放 WebMvcConfigurer即可;继承这个可以
	//3）、自己写一个配置类（在容器中）来继承 WebMvcConfigurationSupport，我们只能去实现模板方法，进行扩展

//这个DelegatingWebMvcConfiguration的特效就是，
	//1、给容器中放组件比如：HandlerMapping
	//2、HandlerMapping的关键环节留给模板方法
	//3、DelegatingWebMvcConfiguration拿到所有的 WebMvcConfigurer，在模板方法实现的时候，
// 由WebMvcConfigurer进行定制
	@Override  //配置视图解析器，升级了这个组件的功能
	public void configureViewResolvers(ViewResolverRegistry registry) {
		registry.viewResolver(new MeiNvViewResolver()); //判断美女
		//完全改变  ，没有 InternalResourceViewResolver


		//不改源码就如下操作
//		InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
//		viewResolver.setPrefix("");
//		viewResolver.setSuffix(".jsp"); //controller的返回值就不用写jsp
//		registry.viewResolver(viewResolver);
		//
	}

	@Override
	public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {

	}
}
```

我们点开这个接口看一下，这个就是里面通过java8定义的默认方法

> 在java8中接口中default方法不需要我们全部实现，我们可以只实现我们需要的方法即可
>
> 比如这里我们实现了configureViewResolvers ，通过这个方法直接向容器中注册我们的MeinvResolver（）

![image-20220304192210935](https://image.imxyu.cn/file/image-20220304192210935.png)



## @EnableWebMvc原理

我们来看一下具体的原理是什么？

点开这个注解，发现给我们导入了一个类

![image-20220304192528339](https://image.imxyu.cn/file/image-20220304192528339.png)

这个类首先有一个@Autowired注解，首先会拿到容器中所有的WebMvcConfigurer类型的组件

而上面我们的类就是实现了这个接口，因此会被注入到这里面，一起加入到这个集合当中

下面的所有方法，都是调用这些WebMvcConfigurer类型组件的方法，也就是我们自己实现的方法。那么这些方法是怎么被调用的呢？

![image-20220304192623297](https://image.imxyu.cn/file/image-20220304192623297.png)

我们观察到这个类继承了WebMvcConfigurationSupport，我们点进去这个父类看一下

```java
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
```

![image-20220304193142759](https://image.imxyu.cn/file/image-20220304193142759.png)



这个父类中有很多@Bean注解，会给容器中注入组件，这里以HandlerMapping defaultServletHandlerMapping() 这个方法举例

并且这里面调用的方法是留给子类的模板方法，我们看到底下这个方法是空的

![image-20220304193344026](https://image.imxyu.cn/file/image-20220304193344026.png)

接下来我们去子类DelegatingWebMvcConfiguration类中找到这个方法，看一下子类对他的实现

![image-20220304193628581](https://image.imxyu.cn/file/image-20220304193628581.png)



可以看到子类中就是for循环所有注入的WebMvcConfigurer类型的组件，调用它的configureDefaultServletHandling方法，这个方法就是我们自己实现的方法

![image-20220304193806604](https://image.imxyu.cn/file/image-20220304193806604.png)

也就是说我们的子类可以自己实现这些方法，注入我们自己需要的类

![image-20220304194058594](https://image.imxyu.cn/file/image-20220304194058594.png)



我们再来看一个熟悉的requestMappingHandlerMapping，首先第一步就是帮我们创建了RequestMappingHandlerMapping mapping = createRequestMappingHandlerMapping();

接下来这里我们可以在里面自定义拦截器

![image-20220304194650489](https://image.imxyu.cn/file/image-20220304194650489.png)

可以看到里面的方法都会先调一下子类的模板，子类通过实现了这个addInterceptors ，然后调用webMvcConfigurer的也就是我们自定义的拦截器就会加入到容器中

![image-20220304194843999](https://image.imxyu.cn/file/image-20220304194843999.png)

![image-20220304194925574](https://image.imxyu.cn/file/image-20220304194925574.png)



> 之前我们说过dispatcherServlet启动的时候会去创建九大组件（如果组件为空的话）
>
> 此时我们的这个注解import的类就帮我们实现了创建九大组件，同时支持我们自定义组件的注入，dispatcherServlet容器此时判断不为空就不会去用它去创建九大组件了



## 自定义组件后没有了默认的组件

当我们自定义组件注入后，会发现容器中没有了默认的internalViewResolver，这可不太好。因为其他的很多默认的转发和重定向功能还是需要这个解析器的

![image-20220305170917089](https://image.imxyu.cn/file/image-20220305170917089.png)

可以看到源码中定义的，父类中定义的先会调用我们实现的看是否有扩展的，如果有扩展了就不会走下面的添加默认的了。

那么我们既想加默认的又想加自己的怎么办呢？

> 这里为什么是判断容器中有一个组件呢？ name.length==1 呢？是因为这里通过@Bean把这个mvcViewResolver 加入到里面了，所以容器中会有它自己

![image-20220305170217675](https://image.imxyu.cn/file/image-20220305170217675.png)

### 不改源码

一种方法就是我们可以在下面手动加入进去

![image-20220305170525408](https://image.imxyu.cn/file/image-20220305170525408.png)

### 改源码

直接改源码，把原来的逻辑换一下，让它必须放入这个组件

![image-20220305170841858](https://image.imxyu.cn/file/image-20220305170841858.png)



## @EnableWebMvc原理图解

![EnableWebMVC注解原理](https://image.imxyu.cn/file/EnableWebMVC%E6%B3%A8%E8%A7%A3%E5%8E%9F%E7%90%86.jpg)

