---
title: 2、BeanDefinition注册流程
tags:
  - spring源码
categories:
  - spring
  - spring源码
cover: 'https://image.imxyu.cn/file/image-20211205141803169.png'
description: 我们写的Bean定义信息是如何一步一步注册到容器中的？
abbrlink: b6c6f05
date: 2021-12-04 21:28:43
---

## BeanDefinition放入档案馆的流程

### debug位置

上一节我们说到了DefaultListableBeanFactory是我们的档案馆（作为BeanDefinitionRegistry的实现类），存储了所有的**bean定义信息**，并且按照每个类名存储到了一个Map中

![image-20211203205002604](https://image.imxyu.cn/file/image-20211203205002604.png)



DefaultListableBeanFactory实现了BeanDefinitionRegistry接口，重写了它的registerBeanDefinition()方法

![image-20211203205535727](https://image.imxyu.cn/file/image-20211203205535727.png)

下面我们就看一下Bean定义信息是如何存储到这个map中的，我们在这个map.put()的时候打一个断点来一步步看



![image-20211203205458051](https://image.imxyu.cn/file/image-20211203205458051.png)



### 测试类

**beans.xml:**

```xml

<bean class="com.atguigu.spring.bean.Person" id="person">
		<property name="name" value="张三"/>
	</bean>

```



```java

public class MainTest {

	public static void main(String[] args) throws IOException {
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
		Person bean = context.getBean(Person.class);
		System.out.println(bean);

	}
}
```





![image-20211207200551112](https://image.imxyu.cn/file/image-20211207200551112.png)

### 首先创建Bean工厂

当new的时候就开始注册Bean定义信息

![image-20211217191251938](https://image.imxyu.cn/file/image-20211217191251938.png)

调用重载的构造函数

![image-20211217191621000](https://image.imxyu.cn/file/image-20211217191621000.png)

刷新容器

![image-20211217191700991](https://image.imxyu.cn/file/image-20211217191700991.png)

开始创建容器

![image-20211217191800272](https://image.imxyu.cn/file/image-20211217191800272.png)

**刷新**整个bean工厂，spring把造工厂叫刷新工厂

![image-20211217191901883](https://image.imxyu.cn/file/image-20211217191901883.png)

在刷新工厂的方法中创建了BeanFactory——DefaultListableBeanFactory 也就是我们的**档案馆**，之前说过我们用的ClassPathXmlApplicationContext这个ioc容器其实就是其父类**组合**了DefaultListableBeanFactory 的，我们真正使用的还是档案馆，可以看到这里把档案馆建了出来。接着开始加载**Bean定义信息**(这里定义信息就是我们写的xml)

![image-20211217194912917](https://image.imxyu.cn/file/image-20211217194912917.png)

### 加载Bean定义信息

#### 资源转换成Resource

然后创建了**XmlBeanDefinitionReader**， XML定义信息读取器，而该读取器组合了**ResourceLoader**，在这里设置了ResourceLoader为this,this对象就是我们当前调用的ClassPathXmlApplicationContext

![image-20211217195254189](https://image.imxyu.cn/file/image-20211217195254189.png)

看继承图可以知道我们用的ClassPathXmlApplicationContext也是ResourceLoader

![image-20211217201213584](https://image.imxyu.cn/file/image-20211217201213584.png)

获取所有的配置文件的**路径**，得到一个String的Location[ ]数组

![image-20211217201724219](https://image.imxyu.cn/file/image-20211217201724219.png)

可以看到我们最开始new **ClassPathXmlApplicationContext**的时候是一次传入多个配置文件的，这里接收的是一个**String数组**

![image-20211217201834968](https://image.imxyu.cn/file/image-20211217201834968.png)

接下来for循环加载每个配置文件的信息，并且返回配置文件的个数进行统计

![image-20211217202120492](https://image.imxyu.cn/file/image-20211217202120492.png)

![image-20211217202342380](https://image.imxyu.cn/file/image-20211217202342380.png)

之前说过XmlBeanDefinitionReader中设置了资源加载器也就是this，这里获取资源加载器开始对每个配置文件进行转换成spring定义的Resource

![image-20211217203024042](https://image.imxyu.cn/file/image-20211217203024042.png)

#### resource封装

同时在这里loadBeanDefinitions（resources）的时候将其包装了一下

![image-20211217203709271](https://image.imxyu.cn/file/image-20211217203709271.png)

这里使用了适配器/包装模式，getReader的时候实际返回的InputStreamReader

![image-20211217204310151](https://image.imxyu.cn/file/image-20211217204310151.png)

![image-20211217204323957](https://image.imxyu.cn/file/image-20211217204323957.png)

#### resource解析成doc

接下来在类XMLBeanDefinitionReader中利用dom解析工具，将resource解析成document对象

![image-20211217204754344](https://image.imxyu.cn/file/image-20211217204754344.png)

#### 解析document对象

随后使用BeanDefinitionDocumentReader 将document对象解析

![image-20211217204939578](https://image.imxyu.cn/file/image-20211217204939578.png)

实际BeanDefinitionDocumentReader 中是使用这个解释器来进行解析的，这个解释器中存储了每个标签的名字，判断读到任意标签的操作

root是整个document的根节点

![image-20211217205444256](https://image.imxyu.cn/file/image-20211217205444256.png)

* document解析是将xml一次读取完封装成一个Document对象，然后你从对象中拿Definition

* SAX解析是一行一行的读取,每一行都封装成Definition对象

遍历文档中的所有节点

![image-20211217205655014](https://image.imxyu.cn/file/image-20211217205655014.png)



这里解析到Bean标签

![image-20211217205812166](https://image.imxyu.cn/file/image-20211217205812166.png)





#### 解析doc后生成BeanDefinitionHolder

解析完的标签都使用BeanDefinitionHolder装起来，然后使用工具类进行注册

![image-20211217210318294](https://image.imxyu.cn/file/image-20211217210318294.png)



BeanDefinitionHolder这个类包含了Bean定义信息和bean的名字以及别名

![image-20211217210052829](https://image.imxyu.cn/file/image-20211217210052829.png)

### BeanDefinition注册到档案馆

随后进入注册的流程

1、先在BeanDefinitionHolder中获取封装的Bean定义信息，进行注册

2、如果有别名，再将别名注册进去

![image-20211217210521234](https://image.imxyu.cn/file/image-20211217210521234.png)



注册的流程

![image-20211217211451500](https://image.imxyu.cn/file/image-20211217211451500.png)

所有BeanDefinition和Bean的名字都放入各自相应的map

![image-20211217211533085](https://image.imxyu.cn/file/image-20211217211533085.png)

## BeanDefinition创建的过程

### debug位置

我们之前说过通过**配置类**的@Import注解可以传入一个自定义的类，该类我们可以自己向容器中注册组件

![image-20211217214546593](https://image.imxyu.cn/file/image-20211217214546593.png)

这个是我们实现的自定义类，手动创建Bean定义信息，并且指定类型，加入到档案馆

![image-20211217214710513](https://image.imxyu.cn/file/image-20211217214710513.png)

我们可以在BeanDefinition的构造函数中打一个断点，看何时创建的Bean定义信息

注册的时候构造函数中实际是new了他的父类，没有调用它

![image-20211217214752179](https://image.imxyu.cn/file/image-20211217214752179.png)



我们直接给它的父类打断点，为了防止它上来直接new多个参数的构造，给它所有参数的构造函数都打上断点

![image-20211217214917554](https://image.imxyu.cn/file/image-20211217214917554.png)

### 根据id和class创建BeanDefinition

这次断点进入到Bean定义信息的创建的步骤

下面的这个方法就是我们之前看的创建完了**注册Bean定义信息**的过程

![image-20211217215816138](https://image.imxyu.cn/file/image-20211217215816138.png)

接下来获取标签中的id属性和名字属性，这里可以看到最后默认bean的名字就是我们写的id值

![image-20211217215957258](https://image.imxyu.cn/file/image-20211217215957258.png)

获取classname属性，然后开始创建bean定义信息

![image-20211217221543910](https://image.imxyu.cn/file/image-20211217221543910.png)

如果标签中指定了parent还会将parent传进来

![image-20211217220153611](https://image.imxyu.cn/file/image-20211217220153611.png)

直接new了一个BeanDefinition来封装标签中的内容![image-20211217220342075](https://image.imxyu.cn/file/image-20211217220342075.png)

调用父类的构造

![image-20211217220454369](https://image.imxyu.cn/file/image-20211217220454369.png)





![image-20211217220504170](https://image.imxyu.cn/file/image-20211217220504170.png)



这就是将id和class转换成BeanDefinition对象的过程

![image-20211217220648983](https://image.imxyu.cn/file/image-20211217220648983.png)

### 设置Bean标签内部的其他信息

先根据i**d和class创建好这个BeanDefintion后**，设置一些Bean标签里面的信息![image-20211217220823995](https://image.imxyu.cn/file/image-20211217220823995.png)



## BeanDefinition注册流程图

![image-20211205141803169](https://image.imxyu.cn/file/image-20211205141803169.png)

