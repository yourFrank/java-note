---
title: 1、Spring整体流程
tags:
  - spring源码
categories:
  - spring
  - spring源码
cover: >-
  https://image.imxyu.cn/file/Spring%E6%9E%B6%E6%9E%84%E5%8E%9F%E7%90%86%E5%9B%BE.jpg
description: 先大体看一下spring的整体脉络，后面看源码的时候知道每一步走到哪了
abbrlink: 5185347b
date: 2021-12-01 21:28:43
---
## spring整体流程

> 此处省略spring源码搭建过程，笔者采用gitee镜像github仓库，使用官方推荐的gradle搭建。此处为了gradle版本一致性，不建议使用本地gradle仓库



spring ioc最简单来说就是读取我们的配置文件/注解，利用反射创建对象，保存到容器中，我们需要的时候从中获取，下面我们看一下整个过程详细图

这个图后期要印在我们脑海里，debug源码的时候要对应图中看现在走到了哪一步了

<img src="https://image.imxyu.cn/file/Spring%E6%9E%B6%E6%9E%84%E5%8E%9F%E7%90%86%E5%9B%BE.jpg" alt="Spring架构原理图"  />



* 基础接口：
  *  Resource ：资源(我们写的**xml**文件或者**注解**。可以在任意位置的资源，类路径、网络磁盘或者网上路径都支持)
  *  ResourceLoader：资源加载器可以根据路径得到相应的资源Resource
  * BeanFactory  ：Spring中的最大工厂（最外面那层大的）
  * Beandifinition：Bean的定义信息（XML底层会对应一个Beandifinition）
  * BeanDefinitionReader：将Resource（XML或注解）转换成doucument，再通过BeanDefinitionDocumentReader将document转成BeanDefinition存到BeanDefinitionRegistry档案馆中
  * BeanDefinitionRegistry接口： 档案馆只存BeanDefinition （Bean的定义信息）
  * ApplicationContext
  * Aware



下面我们来对这几个基础接口进行简要的分析：

## 核心接口类



### Resources

所有的资源(可以是xml文件，可以是注解,网络磁盘,url地址.....)。   ResourceLoader：加载各种资源

了解即可：java中有Resource，spring为何要自己创建Resource？

> 1、在java中将不同来源的资源抽象成**URL**，通过注册不同的handler处理不同资源的逻辑，URL只有简单的如："file:" "http:" "jar:"等，**然而没有Classpath（类路径下）或ServeletContext等资源的handler**，虽然可以通过自定义注册handler来实现，但是需要了解URL的实现机制，实现起来需要前置知识
>
> 2、并且URL没有提供基本的方法，如检查当前资源是否存在、检查当前资源是否合法可读等。

因此spring创建了自己的Resource ，并且提供了相应的资源判定方法

![image-20211101201833095](https://image.imxyu.cn/file/image-20211101201833095.png)

Resource相应的实现类

![image-20211201200510768](https://image.imxyu.cn/file/image-20211201200510768.png)

### ResourceLoader

用来加载不同位置的URL路径，将其转换成资源Resource。

![image-20211101203139141](https://image.imxyu.cn/file/image-20211101203139141.png)

<img src="https://image.imxyu.cn/file/image-20211201201355774.png" alt="image-20211201201355774"  />



ResourceLoader.getResource（）方法   **因为根据路径的不同，会有不同的资源加载方法**，因此这里使用了**策略模式**（对于不同的情况下有不同的算法，我们要根据实际情况切换不同的算法，环境类- 策略接口-策略实现）。如下图：



![image-20211027213828749](https://image.imxyu.cn/file/image-20211027213828749.png)

Resourceloader的实现类：

* ClassRelatvieResourceLoader:加载类相对路径的

* FileSystemResourceLoader:加载文件系统路径的

* ServletContextResourceLoader:加载web容器路径的 

  ​	 .......

> ResourceLoader是我们的策略接口，而AbstractApplication 则是环境类持有该接口，根据不同的资源路径使用不同的loader进行加载。下面马上讲到

### BeanFactory

Spring最大的工厂：是工厂方法模式，没有其他产品线，只有Bean

对于工厂模式的讲解：[工厂模式](https://imxyu.cn/javadoc/#/docs/design_patterns/creational/%E5%88%9B%E5%BB%BA%E5%9E%8B-%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F)

![image-20211203201244129](https://image.imxyu.cn/file/image-20211203201244129.png)

根据spring注释我们可以知道，它是唯一的**根接口**，整个 容器访问的接口，保存很多的BeanDefinition信息，每个Bean定义信息都有一个唯一的名字。可以利用原型模式返回多例Bean,利用单例模式返回单例Bean 。

我们来看一下BeanFactory的子类

> 根据show diagarm显示出BeanFactory类图，show implemetions时先找出**所有实现的接口**，再往下找子类

![image-20211217200904257](https://image.imxyu.cn/file/image-20211217200904257.png)

BeanFactory的子类

- HierarchicalBeanFactory：定义工厂父子关系的（父 子容器）

- ListableBeanFacotory：有获取所有BeanDefinition信息的名字、数量等方法.. 实现类是DefaultListableBeanFactory，保存了ioc容器中的核心信息
- AutowireCapableBeanFactory：提供自动装配能力

#### ListableBeanFacotory的实现类

![image-20211101213717377](https://image.imxyu.cn/file/image-20211101213717377.png)

##### AbastactApplicationContext

资源加载器ResourceLoader的环境类

![image-20211101212435030](https://image.imxyu.cn/file/image-20211101212435030.png)

该类组合了ResourcePatternResolver（策略接口），并且在初始化的时候就注入了ResourcePatternResolver，而该类正是ResourceLoader的子类，因此AbstractApplication就是我们所说的环境类，持有了策略的接口，并且可以在构造函数的时候就传入相应的实现

![image-20211101212613398](https://image.imxyu.cn/file/image-20211101212613398.png)

##### DefaultListableBeanFactory

该类保存了Bean的所有定义信息（档案馆）

![image-20211101213802133](https://image.imxyu.cn/file/image-20211101213802133.png)

我们之前说过将XML转换为Bean定义信息，将Bean的定义信息就存储到档案馆（BeanDefinitionRegistry）这是个接口，而DefaultListableBeanFactory类实现了这个接口，真正就存储在这里。

我们使用的AnnotationConfigApplication/ClassPathXMlApplication的父类GenericApplicationContext组合了档案馆

![image-20211203203855834](https://image.imxyu.cn/file/image-20211203203855834.png)



根据定义信息（图纸）进行创建对象

。。。。



我们首先来了解了整个流程，可能说到这里你有点懵，没关系。我们掌握了大体流程之后再看每一步是怎么做的

