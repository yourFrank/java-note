

## HandlerMapping 寻找处理器映射器

上面我们说完了八大组件的初始化流程，此时我们已经完成了初始化的工作，我们可以专心回到

doDisptatch方法中来

在判断完文件处理请求之后，会对我们的请求寻找相应handler

我们来进入这个方法来看一下

![image-20220220144136666](https://image.imxyu.cn/file/image-20220220144136666.png)

这个方法中首先获取所有的handlerMappings

这三个就是我们之前初始化的时候放的三个

![image-20220220144720045](https://image.imxyu.cn/file/image-20220220144720045.png)

我们来简单介绍一个这三个handlerMapping

![image-20220220144937609](https://image.imxyu.cn/file/image-20220220144937609.png)

接下来我们来看一下它是怎么找的

每一个handlerMappings中都保存了路径和对应的handler(controller)，都保存在一个map中

默认用RequestMappingHandlerMapping ，因此我们来看这个是怎么 找到的我们路径

![image-20220220145714585](https://image.imxyu.cn/file/image-20220220145714585.png)

调用getHandlerInternal（）

![image-20220220145840814](https://image.imxyu.cn/file/image-20220220145840814.png)



这里的lookPath就是/hello ,然后传入request请求。开始查找

![image-20220220145756527](https://image.imxyu.cn/file/image-20220220145756527.png)

所有的请求和处理都保存在了这个mappingRegistry中，这里直接获取就可以得到。 

因此我们看一下这些路径和handler是什么时候放进去的？

![image-20220220151735822](https://image.imxyu.cn/file/image-20220220151735822.png)



## mappingRegistry何时注册所有的请求

ctrl+F ，搜索这个mappingRegistry，我们可以找到在这里的注册流程

发现两个注册的流程

![image-20220221192406034](https://image.imxyu.cn/file/image-20220221192406034.png)

![image-20220221192416115](https://image.imxyu.cn/file/image-20220221192416115.png)

这两个都是调用的this.mappingRegistry.register(mapping, handler, method);，因此我们给里面打一个断点

下面我们就来看一下是什么时候放入的这些请求？

我们给这个方法打一个断点，，里面还有一个registry.put() ，我们给这个方法打一个断点

开始debug，看什么时候put的

![image-20220221192541416](https://image.imxyu.cn/file/image-20220221192541416.png)

我们看一下这个栈的流程

![image-20220221192654351](https://image.imxyu.cn/file/image-20220221192654351.png)

前面的步骤我们之前分析过了

就是容器初始化、spring发送事件，回调到disptacherServlet的onRefresh方法初始化九大组件

然后创建handlerMapping这个组件的时候使用了默认值，创建了3个handlerMapping组件，我们默认使用的是RequestMapping。

> 我们想一下肯定是扫描了所有的controller， 然后找到@RequestMapping的注解，将他们之间的映射关系保存了下来
>
> 下面我们具体看一下

### HandlerMapping放入数据的步骤概括

![image-20220221194419125](https://image.imxyu.cn/file/image-20220221194419125.png)





### HandlerMapping放入数据的源码

我们来看一下步骤4.3之后的，

在创建对象的时候（这里面调用的就是lambda表达式创建bean的过程），组件创建赋值完成后进行了组件初始化方法？

![image-20220221194841926](https://image.imxyu.cn/file/image-20220221194841926.png)

可以看到这个RequestMappingHandlerMapping也是一个initializingBean

![image-20220221195016436](https://image.imxyu.cn/file/image-20220221195016436.png)

调用组件自己实现的afterPropertiesSet（）方法

![image-20220221195117227](https://image.imxyu.cn/file/image-20220221195117227.png)

![image-20220221195253648](https://image.imxyu.cn/file/image-20220221195253648.png)

接下来获取了所有的getCandidateBeanNames()。获取所有的beanNames for循环进行处理判断

> 这里可以看到spring聪明的只获取了web容器（子容器）中的bean，并没有service那些。也就是说父容器中的组件没有



![image-20220221195240765](https://image.imxyu.cn/file/image-20220221195240765.png)

这里就是处理判断的逻辑，首先会对没一个bean进行类型的判断

![image-20220221195718149](https://image.imxyu.cn/file/image-20220221195718149.png)

如果是@controller或者@requestMapping注解的进入detectHandlerMethods(beanName); 方法中

> 也就是说，我们的类需要标注@Controller或者@Component+@RequestMapping  其中Component要被web容器扫描进入
>
> 满足这两种任意一个才会进入处理逻辑

![image-20220221195654917](https://image.imxyu.cn/file/image-20220221195654917.png)



其中getMappingForMethod，获取了所有标注了 RequestMapping注解的方法

![image-20220221195452645](https://image.imxyu.cn/file/image-20220221195452645.png)

这里找到 了三个这样的方法，然后进行注册进去

![image-20220221200146746](https://image.imxyu.cn/file/image-20220221200146746.png)

![image-20220221200223230](https://image.imxyu.cn/file/image-20220221200223230.png)

最终来到了我们的registry.put 方法，将handler和对应的方

![image-20220221200236048](https://image.imxyu.cn/file/image-20220221200236048.png)

这个registry最终保存了这三个mapping和对应的Handler方法

![image-20220221200419436](https://image.imxyu.cn/file/image-20220221200419436.png)

## 得到相应的handler处理器构造成链

知道了handlerMapping是什么时候放入数据的了，我们继续回到刚刚的处理器获取对应的handlerMapping的过程

这里在这个RequestMappingHandlerMapping处理器映射器中找到了能处理这个路径的handler

接着要构造一个处理器链

![image-20220221202306524](https://image.imxyu.cn/file/image-20220221202306524.png)

构造处理器链就是拿到系统中所有的interceptor，构造成一个链

![image-20220221202359681](https://image.imxyu.cn/file/image-20220221202359681.png)

这里直接返回了一个链（目标方法+所有的拦截器）

![image-20220221201846761](https://image.imxyu.cn/file/image-20220221201846761.png)

![image-20220221202602797](https://image.imxyu.cn/file/image-20220221202602797.png)

下一节我们来看handlerAdapter适配器的过程

这两个都是mvc的重要组件

