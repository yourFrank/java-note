接下来我们来探索一下容器刷新的十二大步

## debug大招

因为我们之前说过，每一个对象要创建，肯定要先创建定义信息，所以我们可以给RootBeanDefinition的构造器打一个断点

![image-20220105191018490](https://image.imxyu.cn/file/image-20220105191018490.png)

而这个构造器继承了父类，我们直接在父类的所有构造器打个断点

![image-20220105191401246](https://image.imxyu.cn/file/image-20220105191401246.png)

> 这里就是我们说的大招,因为所有组件的创建都会经过这个构造器,我们给这个构造器打了断点,然后debug,就可以看到每个组件的创建过程!!

使用注解类探索

![image-20220105191603440](https://image.imxyu.cn/file/image-20220105191603440.png)

## 容器启动过程

### this()几个核心后置处理器定义信息注册

首先断点来到了这里

![image-20220105191646131](https://image.imxyu.cn/file/image-20220105191646131.png)

我们来看一下，是如何进入的

![image-20220105191729954](https://image.imxyu.cn/file/image-20220105191729954.png)

调用new 配置类的构造函数

首先进入调用this()无参构造

![image-20220105191736136](https://image.imxyu.cn/file/image-20220105191736136.png)

为了读取配置类中的BeanDefinition，首先需要准备一个Reader读取器，这里把this自己(AnnotationConfigApplicationContext)传了进去

![image-20220105191829699](https://image.imxyu.cn/file/image-20220105191829699.png)

因为我们使用的AnnotationConfigApplicationContext，既是工厂，也是图纸的中心（BeanDefinitionRegistry）

![image-20220105192044817](https://image.imxyu.cn/file/image-20220105192044817.png)

继续传

![image-20220105192342390](https://image.imxyu.cn/file/image-20220105192342390.png)

![image-20220105192404066](https://image.imxyu.cn/file/image-20220105192404066.png)

首先获取获取当前registry的DefaultListableBeanFactory

然后添加了一个以来比较器和一个自动注入候选器

接下来开始放各种后置处理器的**定义信息**，方便后面在合适的时刻调用相应的处理器。（最后流程图总结中写出来几个后置处理器）

> 像ConfigurationClassPostProcessor配置文件处理的后置处理器，AutowiredAnnotationBeanPostProcessor自动装配的后置处理器等都是在这里加进来的
>
> 注意：这里加的都是BeanDefinition **图纸**，还没有创建后置处理器对象

![image-20220105193826961](https://image.imxyu.cn/file/image-20220105193826961.png)

![image-20220105193932681](https://image.imxyu.cn/file/image-20220105193932681.png)

放置各种处理器定义信息（图纸），比如这里点进去jpa的后置处理器过程

![image-20220105194351867](https://image.imxyu.cn/file/image-20220105194351867.png)

可以看到首先会获取类加载器，然后下面会判断是否引入了相关的jar包，如果引入了jsr250Present设置为true，然后才将该后置处理器加入到BeanDefinition

![image-20220105194426850](https://image.imxyu.cn/file/image-20220105194426850.png)

可以看到，经过一系列判断之后，我们这里加入了这4个后置处理器的Bean定义信息

其中包括解析配置类的、自动注入的的后置处理器、还有事件相关的两个（这个后面会说到）

> 注意这里都是图纸，还没有真正的创建对象

![image-20220105194936539](https://image.imxyu.cn/file/image-20220105194936539.png)

这样，就Reader就准备好了，这个过程中注册了很多后置处理器，

然后又开始准备扫描器

![image-20220105195238371](https://image.imxyu.cn/file/image-20220105195238371.png)

扫描器里面就是设置了一些环境变量信息，我们直接放行

![image-20220105195402219](https://image.imxyu.cn/file/image-20220105195402219.png)

这样this（）构造函数中的任务就结束了

### register()配置类定义信息注册

然后进入了regiter函数调用,这里把我们的配置类传了进去

![image-20220105195618689](https://image.imxyu.cn/file/image-20220105195618689.png)

接下来reader会注册我们写的MainConfig,下面我们看看他的注册流程

![image-20220105195714876](https://image.imxyu.cn/file/image-20220105195714876.png)

这里有个for循环注册,传了几个配置类就会注册几个

![image-20220105195806864](https://image.imxyu.cn/file/image-20220105195806864.png)

调用注册

![image-20220105195833588](https://image.imxyu.cn/file/image-20220105195833588.png)

可以看到是在这里进入了我们的断点,也就是说在这里创建了**配置类的定义信息**

创建完成后,processCommonDefinitionAnnotations()调用这个方法来完善配置类定义信息,我们点进去这个方法看看

![image-20220105200424100](https://image.imxyu.cn/file/image-20220105200424100.png)

这个方法会将所有标注了这些关键注解的地方都加入到Bean定义信息中,完善一波配置类的定义信息

![image-20220105200552353](https://image.imxyu.cn/file/image-20220105200552353.png)

加入到图纸库

到此为止就完成了几个核心后置处理器和配置类的**定义信息**的创建

![image-20220105200732976](https://image.imxyu.cn/file/image-20220105200732976.png)

> 这一步register()就多了配置类的定义信息，那4个后置处理器的定义信息是上一步this()的时候创建的

接下来就是refresh(), 刷新容器的过程



### refresh()

下面就是容器完整刷新，创建出所有组件，组织好所有功能

我们来看refresh()中

![image-20220106194641132](https://image.imxyu.cn/file/image-20220106194641132.png)

接下来进入模板十二大步

![image-20220106195227948](https://image.imxyu.cn/file/image-20220106195227948.png)

#### 1.prepareRefresh()准备环境

prepareRefresh（）首先开始准备上下文环境，我们点进去看看

首先会保存系统的当前时间，然后设置一些变量，initPropertySources（）这里预留了一个模板方法可以子类实现

下面就是容器监听器事件List创建，一开始是没有监听器的

我们点进去initPropertySources（）看一下

![image-20220106195359118](https://image.imxyu.cn/file/image-20220106195359118.png)

```java
	/**
	 * <p>Replace any stub property sources with actual instances.
	 * @see org.springframework.core.env.PropertySource.StubPropertySource
	 * @see org.springframework.web.context.support.WebApplicationContextUtils#initServletPropertySources
	 */
	protected void initPropertySources() {
		// For subclasses: do nothing by default. 自行在此处加载一些自己感兴趣的信息。【WebApplicationContextUtils.initServletPropertySources】web-ioc容器启动的时候一般在此加载当前应用的上下文信息（ApplicationContext）
	
   }
```

可以看到这个方法里面是空的，这个是模板方法，可以留给子类去重写实现的，也就是说我们子类可以可以实现这个方法，到时候调用的我们实现的方法

![image-20220106195625827](https://image.imxyu.cn/file/image-20220106195625827.png)

> 我们可以在这里实现加载任意的信息，web-ioc 容器启动的时候就是在此加载应用的上下文信息ApplicationContext

#### 2.obtainFreshBeanFactory XML解析配置类在此

我们点进第二步看看

![image-20220106200603208](https://image.imxyu.cn/file/image-20220106200603208.png)

1、refreshBeanFactory();

2、getBeanFactory（）; 获取工厂

在这里对于**注解模式**refreshBeanFactory（）就是设置了一下工厂id，而对于xml 则是在这里创建了工厂，解析xml配置文件中的信息放进去

![image-20220106200638476](https://image.imxyu.cn/file/image-20220106200638476.png)

这是注解类的结构图，有一个子类实现了refreshBeanFactory（），设置了一下工厂id

![image-20220106201305145](https://image.imxyu.cn/file/image-20220106201305145.png)

对于XML的方式，这里有一个子类AbstractRefreshableApplicationContext实现了这个方法refreshBeanFactory，在这里会创建工厂，注册XML的bean定义信息

这也是我们之前分析过的

![image-20220106201122931](https://image.imxyu.cn/file/image-20220106201122931.png)

对于配置类，因为我们第一步this() ，就已经创建好了工厂然后将配置类的定义信息加入到工厂了。

这里直接得到this（）时期创建的工厂。这就是配置类中这obtainFreshBeanFactory（）的两步

> 对于配置类目前只是将配置类的定义信息加入到容器中，配置类中定义的组件的定义信息还没有进行解析，在后面第五步的时候会调用后置处理器进行解析

![image-20220106201756107](https://image.imxyu.cn/file/image-20220106201756107.png)

之后所有的操作都是对这个工厂的操作，bean定义信息、bean、后置处理器的存储等

#### 3.prepareBeanFactory(beanFactory)放2个后置处理器和环境信息

prepareBeanFactory（）会给容器注册环境信息放入单例池，方便后续自动装配，还放置了一些后置处理器（监听、XXXAware功能的）

我们点进去看看

![image-20220106203147747](https://image.imxyu.cn/file/image-20220106203147747.png)

首先设置了Bean的解释器，可以解析EL,SPEL表达式的。这里用到了解释器模式

然后放置了一个属性注册器

接下来直接new了一个后置处理器**ApplicationContextAwareProcessor**（this）并且将当前容器的放进去 ，这也是我们之前分析过的可以处理XXXAware接口功能的后置处理器

并且将这几个接口忽略了，告诉spring先别管这几个接口，留给这个后置处理器来处理

然后放了一个**ApplicationListenerDetector**，监听功能的后置处理器

![image-20220106203431964](https://image.imxyu.cn/file/image-20220106203431964.png)

这是我们之前分析过的，可以看到我们之前说的这个ApplicationContextAwareProcessor后置处理器的功能就是这个，可以为我们实现XXXAware接口的注入相应的属性

![image-20220106203925702](https://image.imxyu.cn/file/image-20220106203925702.png)

注册完这两个后置处理器

接下来将这几个组件添加到容器当中，我们的ApplicationContext, BeanFactory，也就是在这里注册到容器中的，

这也就是为什么我们后面也可以通过@Autowired直接获取到他们

![image-20220106204643545](https://image.imxyu.cn/file/image-20220106204643545.png)

resolvableDependencies这个MAP保存了这些信息

![image-20220106204713860](https://image.imxyu.cn/file/image-20220106204713860.png)

![image-20220106204733063](https://image.imxyu.cn/file/image-20220106204733063.png)

然后注册了几个默认组件

环境变量、systemProperties,等信息。

![image-20220106204400240](https://image.imxyu.cn/file/image-20220106204400240.png)

![image-20220106204322322](https://image.imxyu.cn/file/image-20220106204322322.png)

把这些都放到了单例池中

![image-20220106204543500](https://image.imxyu.cn/file/image-20220106204543500.png)

![image-20220106205007535](https://image.imxyu.cn/file/image-20220106205007535.png)

也就是说对于这些注册的组件，我们可以通过@Autowired， 自动注入获取到他们，只要变量名写成他们注册的名字即可

![image-20220106204803381](https://image.imxyu.cn/file/image-20220106204803381.png)

#### 4.postProcessBeanFactory(beanFactory)提供模板允许子类继续对工厂处理

接下来执行postProcessBeanFactory(beanFactory) , 我们点进去看看

![image-20220107181431823](https://image.imxyu.cn/file/image-20220107181431823.png)

发现这个方法是模板方法，留给子类去实现的

可以允许子类对工厂执行一些处理

![image-20220107181536781](https://image.imxyu.cn/file/image-20220107181536781.png)

#### 5.【核心】invokeBeanFactoryPostProcessors执行所有的工厂后置处理器(配置类解析在此)

（核心重点）下面执行工厂中的后置处理器

![image-20220107185933336](https://image.imxyu.cn/file/image-20220107185933336.png)

在这个方法中调用PostProcessorRegistrationDelegate类的方法执行所有的工厂增强器

PostProcessorRegistrationDelegate 类属于一个代理类，所有的bean、beanFactory的后置处理都是由这个类执行的

![image-20220107185950620](https://image.imxyu.cn/file/image-20220107185950620.png)

点进这个类，ctrl+f12 ，查看方法，可以看到。这几个后置处理器都是在这个类帮我们调用的

![image-20220107190249364](https://image.imxyu.cn/file/image-20220107190249364.png)

首先拿到底层默认的beanFactoryPostProcessors，这里没有默认的

![image-20220107190435628](https://image.imxyu.cn/file/image-20220107190435628.png)

下面找实现了BeanDefinitionRegistryPostProcessor接口的组件

1、之前说过，首先找实现了PriorityOrdered 接口的组件

这里找到了一个internalConfigurationAnnotationProcessor ，这也是容器启动过程的this（）第一步里将核心后置处理器注册进去的，我们上面说过

![image-20220107190950036](https://image.imxyu.cn/file/image-20220107190950036.png)

注意此时工厂中的后置处理器的**定义信息**还是只有这几个

![image-20220107191624068](https://image.imxyu.cn/file/image-20220107191624068.png)

对象池中的组件只有这几个

![image-20220107191720725](https://image.imxyu.cn/file/image-20220107191720725.png)

这里拿到了这个internalConfigurationAnnotationProcessor  组件之后，调用了getBean进行创建之后，加入到了currentRegistryProcessors这个lIst集合中

然后调用invokeBeanDefinitionRegistryPostProcessors() 方法来调用list集合中后置处理器的方法，这里集合中只有这一个组件，我们点进来看看

![image-20220107191921811](https://image.imxyu.cn/file/image-20220107191921811.png)

这里会调用这个配置类的后置处理器的方法 postProcessBeanDefinitionRegistry（）

点进去看看

![image-20220107192007024](https://image.imxyu.cn/file/image-20220107192007024.png)

这里会处理配置类的BeanDefinition信息

processConfigBeanDefinitions(registry)，我们点进这个方法看看

![image-20220107192524237](https://image.imxyu.cn/file/image-20220107192524237.png)

在这个方法中，首先拿到所有的bean定义信息，此时包括了后置处理器和mainconfig

然后检查，看哪个是配置类，然后将配置类添加到configCandidates这个list集合中

![image-20220107192917254](https://image.imxyu.cn/file/image-20220107192917254.png)

可以看到这里检查结束后，找到了这个配置类的定义信息，并将它加入到了集合当中

![image-20220107193135187](C:\Users\tianyu\AppData\Roaming\Typora\typora-user-images\image-20220107193135187.png)

接下来如果找到了多个配置类，将他们排序

![image-20220107193304144](https://image.imxyu.cn/file/image-20220107193304144.png)

接下来，一个重要的方法，调用parser解析器来解析配置类中的所有信息，我们点进来看看

![image-20220107194421587](https://image.imxyu.cn/file/image-20220107194421587.png)

省略了几个方法调用

首先会从缓存中查找已经解析的配置类，如果已经解析过了就会放到这个缓存中，下次就不需要再解析了

调用doProcessConfigurationClass() 方法来解析配置类

![image-20220107194715407](https://image.imxyu.cn/file/image-20220107194715407.png)

doProcessConfigurationClass（）方法中就是解析这几个注解

@Component,@ComponentScans,@Import, @Bean 这些注解的。解析完成后将他们的**定义信息**加入到容器中

至此配置类中的各种定义的各种包扫描，将我们需要的Bean的定义信息都加入到容器了

![image-20220107195021086](https://image.imxyu.cn/file/image-20220107195021086.png)

此时容器中的状态，有了这些定义信息

![image-20220107195437471](https://image.imxyu.cn/file/image-20220107195437471.png)

至此解析配置类的这个方法调用就结束了

下面就是来调用执行实现Ordered接口的后置处理器

和什么都没实现的后置处理器的调用。这些我们之前都说过了

![image-20220107195921979](https://image.imxyu.cn/file/image-20220107195921979.png)

整个方法执行完了之后，我们的容器中就会多了几个我们的后置处理器的组件

**但是此时只是将Bean的定义信息通过配置类的后置处理器解析配置类加入到Map图纸库中，还没有开始创建bean组件的过程**

![image-20220107200121341](https://image.imxyu.cn/file/image-20220107200121341.png)

#### 6.registerBeanPostProcessors(beanFactory) 创建所有bean的后置处理器实例注入

下面开始注册所有Bean的后置处理器

这个也是我们之前详细说过的，详细的可以看我们之前讲的

这里粗略的看一下

![image-20220107200350514](https://image.imxyu.cn/file/image-20220107200350514.png)

这里和之前我们说的一样都是按照优先级，将所有的bean后置处理器创建对象，添加到容器

这里注意最后一步，spring还向容器中添加 了一个监听器的后置处理器

这个ApplicationListenerDetector在第三步的时候注册过一次，这里是为了调整顺序，将这个后置处理器放到最后

![image-20220107200606618](https://image.imxyu.cn/file/image-20220107200606618.png)

#### 7.initMessageSource国际化相关

我们点进去看看

![image-20220107201517604](https://image.imxyu.cn/file/image-20220107201517604.png)

首先会看一下bean定义信息中是否有这个messagesource

如果没有spring就自己创建一个默认的，然后加入到单例池中

![image-20220107201806171](https://image.imxyu.cn/file/image-20220107201806171.png)

这个就是spring自己创建的默认的DelegatingMessageSource，里面有spring提供的默认的国际化操作，

后面讲springMVC和springboot中会讲到

![image-20220107201917192](https://image.imxyu.cn/file/image-20220107201917192.png)

同样我们也可以自己实现这个接口自己实现国际化方法，这样上面的步骤就会将我们的这个类的bean定义信息加入到容器中

然后上面就能获取到这个定义信息，就使用我们的

![image-20220107202026290](https://image.imxyu.cn/file/image-20220107202026290.png)

#### 8.initApplicationEventMulticaster初始化事件派发功能

我们但进去看看

![image-20220107202145668](https://image.imxyu.cn/file/image-20220107202145668.png)

这里也是先判断容器中的定义信息中是否有id为**applicationEventMulticaster**的组件

如果没有帮我们new 了一个事件派发器的实例，然后添加到容器中

此时容器中就有了这个名字的事件派发器，我们如果想使用注解注入的话，记得要使用applicationEventMulticaster这个作为名字的

![image-20220107202400996](https://image.imxyu.cn/file/image-20220107202400996.png)

判断容器中是否有这个实例或者Bean定义信息中是否有这个

![image-20220107202441517](https://image.imxyu.cn/file/image-20220107202441517.png)

#### 9.onRefresh() 模板方法给子类

![image-20220107202743098](https://image.imxyu.cn/file/image-20220107202743098.png)

这个方法同样是空的，是一个模板方法，留给子类去自己实现的

此时这里容器中的定义信息，还有几个后置处理器组件、环境信息、和事件派发器国际化等，都有了，我们可以在这里做一些我们想要的操作

![image-20220107202747364](https://image.imxyu.cn/file/image-20220107202747364.png)

#### 10.registerListeners()注册监听器

我们点进去看看

![image-20220107202912859](https://image.imxyu.cn/file/image-20220107202912859.png)

在这里首先获取了所有的监听器，然后将这些监听器和他们的名字加入到多播器(事件派发器)的集合中，事件派发器中有单独一个集合用来保存这些监听器

![image-20220107203854820](https://image.imxyu.cn/file/image-20220107203854820.png)

multicastEvent（）方法 可以派发早期的一些事件给所有的监听器 

![image-20220107203835210](https://image.imxyu.cn/file/image-20220107203835210.png)

派发过程：获取所有的监听器，for循环调用他们的监听事件

这里就是利用了观察者模式

![image-20220107203757287](https://image.imxyu.cn/file/image-20220107203757287.png)



#### 11.【核心】finishBeanFactoryInitialization(beanFactory) Bean创建

![image-20220107204038246](https://image.imxyu.cn/file/image-20220107204038246.png)

这里首先会添加几个组件

```
我们主要看这个方法beanFactory.preInstantiateSingletons();我们点进来看看
```

![image-20220107204310875](https://image.imxyu.cn/file/image-20220107204310875.png)

获取所有的bean定义信息，看还有哪些是没有创建对象的，在这里创建对象。这里区分了如果是工厂bean和普通bean

这里就不详细看了，可以参考上一节详细的bean的生命周期

![image-20220107204600596](https://image.imxyu.cn/file/image-20220107204600596.png)

bean创建后执行所有实现了SmartInitializingSingleton接口的post-initialization方法

![image-20220107204716250](https://image.imxyu.cn/file/image-20220107204716250.png)

#### 12.finishRefresh()发布事件

点进来看看

![image-20220107210849413](https://image.imxyu.cn/file/image-20220107210849413.png)

1、首先就是clearResourceCaches（） 清理一些缓存

2、initLifecycleProcessor（），我们点进来看看

![image-20220107211011614](https://image.imxyu.cn/file/image-20220107211011614.png)

还是和事件派发器和国际化器一样，首先会进行判断是否有该接口

如果没有的话会给你创建一个默认的，我们来看一下这个接口

![image-20220107211240780](https://image.imxyu.cn/file/image-20220107211240780.png)

这个接口中含有这两个方法



![image-20220107211312010](https://image.imxyu.cn/file/image-20220107211312010.png)

接下来调用initLifecycleProcessor接口onRefresh（） 方法

![image-20220107211532397](https://image.imxyu.cn/file/image-20220107211532397.png)

然后就是一个发布事件，调用多播器，这也是我们之前看过的

![image-20220107211643826](https://image.imxyu.cn/file/image-20220107211643826.png)

![image-20220107211715587](https://image.imxyu.cn/file/image-20220107211715587.png)

```
最后一步，执行完这一步我们就能在jconsole控制台看到spring的所有东西，这里展开讲有很多涉及jvm的
//jconsole（暴露MBean端点信息） Participate in LiveBeansView MBean, if active.
if (!NativeDetector.inNativeImage()) {
   LiveBeansView.registerApplicationContext(this);
}
```

最后十二大步finnaly{}中会清除一些缓存，

完结撒花~

![image-20220107211904150](https://image.imxyu.cn/file/image-20220107211904150.png)

## 容器刷新流程图

![容器刷新流程](https://image.imxyu.cn/file/%E5%AE%B9%E5%99%A8%E5%88%B7%E6%96%B0%E6%B5%81%E7%A8%8B.jpg)