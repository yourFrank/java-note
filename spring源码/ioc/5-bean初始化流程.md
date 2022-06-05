接下来我们来看一下bean的初始化流程

## bean初始化流程

我们来看一下bean是如何创建初始化的

![image-20220103155416311](https://image.imxyu.cn/file/image-20220103155416311.png)

主要是这里初始化非懒加载单实例bean的过程

![image-20220103155451320](https://image.imxyu.cn/file/image-20220103155451320.png)

for循环创建bean的过程我们来打个断点看一下

![image-20220103155527243](https://image.imxyu.cn/file/image-20220103155527243.png)

然后使用注解开启debug

![image-20220103155654913](https://image.imxyu.cn/file/image-20220103155654913.png)

![image-20220103160654295](https://image.imxyu.cn/file/image-20220103160654295.png)

 1、首先会判断是否是非抽象的、单实例的、非懒加载的，只有满足了这三个条件才会创建

2、接着判断是否是工厂bean，我们首先来看一下如果是工厂bean

![image-20220103161048119](https://image.imxyu.cn/file/image-20220103161048119.png)

### 工厂bean创建流程

准备了一个FactoryBean

![image-20220103161313906](https://image.imxyu.cn/file/image-20220103161313906.png)

并且我们给这个断点右键进行判断，获取到我们实现的factoryBean

![image-20220103161549703](https://image.imxyu.cn/file/image-20220103161549703.png)

我们点进去看看它是怎么判断是工厂bean的

1、首先会getSingleton（）从单例池中拿，如果之前创建过对象，得到不是null会直接判断类型是不是FactoryBean

没创建的bean会走下面的方法isFactoryBean判断

![image-20220103161616500](https://image.imxyu.cn/file/image-20220103161616500.png)

2、这里判断RootBeanDefinition，之前存bean定义信息的时候会保存这个字段，是否是factorybean

如果result为null没保存的话，会调用predictBeanType判断是否是FactoryBean类型

![image-20220103162030288](https://image.imxyu.cn/file/image-20220103162030288.png)

判断完成后，如果是工厂bean，会调用getBean, 参数是**&+beanName**

> 这就是工厂bean和普通bean的区别，前面会加一个&符号

![image-20220103162719994](https://image.imxyu.cn/file/image-20220103162719994.png)

注意：

上面如果是工厂bean，这里只是创建了工厂bean，而没有创建我们实际想用的bean组件。也就是说容器中不会立马有我们想要用的bean

#### 探究工厂bean何时创建实际的组件

我们继续打一个断点来看看什么时候创建我们实际想用的hello组件

![image-20220103163859150](https://image.imxyu.cn/file/image-20220103163859150.png)

我们获取一下hello组件，我们看一下是怎么创建的

![image-20220103164310629](https://image.imxyu.cn/file/image-20220103164310629.png)

我们根据栈一步一步看，首先是调用根据类型获取

![image-20220103164955782](https://image.imxyu.cn/file/image-20220103164955782.png)



可以看到此时是没有创建Hello组件的

![image-20220103165317371](https://image.imxyu.cn/file/image-20220103165317371.png)

继续调用

![image-20220103165042617](https://image.imxyu.cn/file/image-20220103165042617.png)

![image-20220103165103020](https://image.imxyu.cn/file/image-20220103165103020.png)

接着按照类型去获取组件，我们点进去这个方法看看

![image-20220103165539747](https://image.imxyu.cn/file/image-20220103165539747.png)

继续进去

![image-20220103165633591](https://image.imxyu.cn/file/image-20220103165633591.png)

这里spring还是采用最笨的方法，遍历所有的bean定义信息，挨个判断类型

这里会判断如果是工厂bean的话，下面的isTypeMatch会看这个工厂bean能否产生我们的类型对象

![image-20220103165930944](https://image.imxyu.cn/file/image-20220103165930944.png)

根据类型匹配后找到合适的工厂类，从工厂bean中获取对象，首先会判断是否是单例的（如果我们没设置默认是单例的），如果是单例的首先从工厂bean的缓存中拿，如果没有的话再调用doGetObjectFromFactoryBean（）拿，然后放到缓存中。如果不是单例的，直接工厂产生对象

![image-20220103170931479](https://image.imxyu.cn/file/image-20220103170931479.png)

工厂bean的接口中可以更改是否是单例的

![image-20220103171158447](https://image.imxyu.cn/file/image-20220103171158447.png)

并且最后这个组件不会存到singletonObjects，而是存到factoryBeanObjectCache（）中，下次如果要获取的时候会先在这里拿

![image-20220103171847522](https://image.imxyu.cn/file/image-20220103171847522.png)

我们继续回到刚刚的流程，对于工厂bean我们还可以实现**SmartFactoryBean**接口，可以提前在这里初始化bean

![image-20220103172611886](https://image.imxyu.cn/file/image-20220103172611886.png)

![image-20220103172754979](https://image.imxyu.cn/file/image-20220103172754979.png)

至此工厂bean就创建结束了

### 普通bean的创建流程

我们这里以cat为例

![image-20220103172414519](https://image.imxyu.cn/file/image-20220103172414519.png)

直接来到getBean

![image-20220103173007827](https://image.imxyu.cn/file/image-20220103173007827.png)

继续调用doGetBean

![image-20220103192053514](https://image.imxyu.cn/file/image-20220103192053514.png)

首先会进入getSingleton（）检查缓存中是否有，我们点进去看看。第一次不会走这里，缓存中没有，会走else

![image-20220103192404496](https://image.imxyu.cn/file/image-20220103192404496.png)

点进去看首先在缓存中找

![image-20220103192445311](https://image.imxyu.cn/file/image-20220103192445311.png)

这里会在单例缓存池中找，并且判断是否是正在创建

![image-20220103192928704](https://image.imxyu.cn/file/image-20220103192928704.png)

单例缓存池：

![image-20220103192826565](https://image.imxyu.cn/file/image-20220103192826565.png)

正在创建的保存在singletonsCurrentlyInCreation这个池中

![image-20220103192856902](https://image.imxyu.cn/file/image-20220103192856902.png)

> 这里并不是正在创建，只是在获取的时候进行判断看是否有这个Cat类型对象

这里判断完了，两个都为false，返回了null。此时走了else

![image-20220103193209805](https://image.imxyu.cn/file/image-20220103193209805.png)

接下来拿到父工厂，看父工厂中是否有

> 如果整合了springMVC的话就存在了父子工厂，这里会检查父工厂中有没有

然后markBeanAsCreated（）会看这个bean是否已经被创建，这里是为了防止多线程已经创建了

![image-20220103193603345](https://image.imxyu.cn/file/image-20220103193603345.png)

点进去markBeanAsCreated（），这里其实是一个**备忘录模式**，保存了所有已经创建的bean，这里保存了状态

![image-20220103193845515](https://image.imxyu.cn/file/image-20220103193845515.png)

接下来会调用dependsOn,查看依赖的组件

如果这里有依赖的组件，又调用getBean（）方法去查看依赖的组件

![ ](https://image.imxyu.cn/file/image-20220103193932709.png)

接下来就开始根据单实例或者多实例，或者什么都没有指定的话。这三种情况来创建bean对象了

![image-20220103194150402](https://image.imxyu.cn/file/image-20220103194150402.png)

我们点进去getSingleton（）这个lambda表达式看看

1、首先会在单例池子中看是否有当前对象

2、如果没有，这里首先会调用一个**beforeSingletonCreation**（）方法，我们点进去这个方法看看

![image-20220103195119973](https://image.imxyu.cn/file/image-20220103195119973.png)

然后在创建对象之前首先会在这个**singletonsCurrentlyInCreation**池子中记录这个**beanName**，表示它正在创建

> 这个我们上面说过，调用getBean的时候查完缓存中然后会在这个池子中取进行判断看这个bean是否正在创建

![image-20220103195251837](https://image.imxyu.cn/file/image-20220103195251837.png)

接下来就会调用lambda表达式创建对象

![image-20220103195555513](https://image.imxyu.cn/file/image-20220103195555513.png)

![image-20220103195652613](https://image.imxyu.cn/file/image-20220103195652613.png)

接下来**创建bean**的过程就到了我们之前bean的生命周期的过程

bean的创建、bean的赋值前后的各种生命周期处理器的过程

### createBean生命周期回顾

下面就开始了组件的创建和前后的后置处理器的执行

#### 组件创建前的后置处理器-可以直接返回一个对象

首先进入createBean

在spring创建bean对象之前，预留了一个我们可以自己创建对象的地方resolveBeforeInstantiation（）

![image-20220103201855914](https://image.imxyu.cn/file/image-20220103201855914.png)

点进去看看，这里面有两个后置处理器的方法applyBeanPostProcessorsBeforeInstantiation（）允许我们自己返回一个对象，如果我们这里返回了对象，后面spring就不会为我们创建对象了

如果我们返回了对象 ，我们还可以通过applyBeanPostProcessorsAfterInitialization(bean, beanName)  这个后置处理器的方法来将我们返回的对象修改增强

![image-20220103202014901](https://image.imxyu.cn/file/image-20220103202014901.png)

#### 组件创建中的后置处理器-可以决定构造器

通常情况下我们不会创建返回对象，让spring帮我们创建，于是我们点进spring的**doCreateBean**看一下spring的创建流程

在这里我们能决定使用哪个构造器

![image-20220103203255063](https://image.imxyu.cn/file/image-20220103203255063.png)

如果没有指定，默认使用无参的构造器

![image-20220103203432835](https://image.imxyu.cn/file/image-20220103203432835.png)

#### 组件创建后的后置处理器-允许修改bean定义信息

接下来允许修改bean定义信息

下面的**earlySingletonExposure**很重要，后面会详细讲

![image-20220103203523158](https://image.imxyu.cn/file/image-20220103203523158.png)



随后就是属性赋值、和初始化bean

![image-20220103203607619](https://image.imxyu.cn/file/image-20220103203607619.png)

在属性赋值中，有两个后置处理器可以干预

#### 属性赋值中的后置处理器-允许中断赋值、使用后置处理器处理属性（自动注入在此）

首先给我们预留了一个可以中断赋值的方法，如果我们返回了false,这里后面的赋值操作就结束了

随后得到所有的键值对，开始属性赋值，这里提供了后置处理器，postProcessProperties（）可以对所有的属性进行操作，@Autowired 注解就是在这里进行赋值的

随后就是spring的属性赋值操作

![image-20220103203746068](https://image.imxyu.cn/file/image-20220103203746068.png)

![image-20220103204014089](https://image.imxyu.cn/file/image-20220103204014089.png)

#### 属性赋值后的后置处理器或者实现了InitializingBean接口-可以修改bean

**属性赋值**完了之后，就是exposedObject = initializeBean(beanName, exposedObject, mbd)![image-20220103204122867](https://image.imxyu.cn/file/image-20220103204122867.png)

在这里属性全部赋值完了之后，提供了操作bean的两个后置处理器，可以在这里修改bean



![image-20220103204233723](https://image.imxyu.cn/file/image-20220103204233723.png)

invokeInitMethods中会判断如果组件实现了InitializingBean接口，会调用组件自己的afterPropertiesSet方法

![image-20220103204521401](https://image.imxyu.cn/file/image-20220103204521401.png)

随后会进行三级缓存的一个检查，这个后面会说

![image-20220103205642030](https://image.imxyu.cn/file/image-20220103205642030.png)

还是这个方法，进行缓存的判断，这里是cat正在创建的过程

![image-20220103205715193](https://image.imxyu.cn/file/image-20220103205715193.png)

随后调用registerDisposableBeanIfNecessary，有销毁的流程，这里先不看

于是就返回了exposedObject，创建好的bean

![image-20220103205802863](https://image.imxyu.cn/file/image-20220103205802863.png)

> 至此上面的createBean() 方法的调用就结束了

这里创建完了之后调用afterSingletonCreation（）点进去看看

![image-20220103210015331](https://image.imxyu.cn/file/image-20220103210015331.png)

这里会将正在创建的对象从池子中移除，此时池中的cat就被移除了

我们之前说过这个池子中会保存我们正在创建的bean的名字

![image-20220103210057241](https://image.imxyu.cn/file/image-20220103210057241.png)

随后调用addSingleton（）方法，将对象添加到池中

![image-20220103210204735](https://image.imxyu.cn/file/image-20220103210204735.png)

至此bean创建完成了，然后会判断，这个bean是否是工厂bean创建的，如果是，在这个方法里会进行一些对工厂bean的操作

![image-20220103210625224](https://image.imxyu.cn/file/image-20220103210625224.png)

随后就将object类型转换为T类型返回了

![image-20220103210720992](https://image.imxyu.cn/file/image-20220103210720992.png)

至此上面的for循环创建bean的过程就结束了

![image-20220103210810808](https://image.imxyu.cn/file/image-20220103210810808.png)

#### 添加到容器后执行所有实现了SmartInitializingSingleton接口方法

下面还有一个for循环，对所有的bean进行判断，如果实现了SmartInitializingSingleton接口的，在这里会调用组件实现的**afterSingletonsInstantiated**（）方法

![image-20220103210847200](https://image.imxyu.cn/file/image-20220103210847200.png)

至此这两个for循环就完结了，所有的bean初始化结束

![image-20220103211103652](https://image.imxyu.cn/file/image-20220103211103652.png)

完成

![image-20220103211219235](https://image.imxyu.cn/file/image-20220103211219235.png)

## bean初始化流程图

![bean初始化流程](https://image.imxyu.cn/file/bean%E5%88%9D%E5%A7%8B%E5%8C%96%E6%B5%81%E7%A8%8B.jpg)