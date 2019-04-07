# Spring

## 1 Spring的核心模块：

​	**核心容器(SpringCore)**：提供了Spring框架的基本功能，核心容器的主要组件是**BeanFactory**，他是**工厂模式**的实现。Bean Factory使用控制反转的模式将应用程序的配置和依赖性规范与实际的应用程序代码相分开。

​	**应用上下文(SpringContext)**：是一个配置文件，向Spring模块提供上下文信息。Spring上下文包含了一些企业服务，例如：JNDI、EJB、电子邮件、国际化、校验、调度等功能。

​	**AOP模块(Spring AOP)** ：通过配置管理特性，Spring AOP模块直接将面向切面的编程功能集成到了Spring框架当中。所以，可以很容易的使Spring框架管理的任何对象都可以支持AOP。Spring的AOP模块为基于Spring的应用程序中的对象提供了事物管理功能，通过使用Spring AOP不用依赖EJB组件就可以将**声明性事务**管理集成到应用程序当中。

​	**JDBC和DAO模块(Spring DAO)**：JDBC、DAO的抽象层提供了有意义的异常层次结构,可用该结构来管理异常处理和不同数据库供应商所抛出的异常信息。异常层次结构简化了错误处理,并且极大的降低了需要编写的异常处理代码数量。例如打开和关闭连接等等。SpringDAO的面向切面，JDBC的异常遵从通用的DAO异常层次结构。

​	**对象实体映射（SpringORM）**:Spring框架插入了若干个ORM框架，从而提供了ORM对象的关系工具。其中包括JDO、Hibernate… …所有这些都遵从Spring的通用事物和DAO异常层次结构。

​	**Web模块(Spring Web)**：Web上下文模块建立在应用程序上下文模块之上,为基于Web的应用程序提供了上下文，所以Spring框架支持与Struts的集成。Web模块还简化了处理多部分请求以及将请求参数绑定到预对象的操作。

​	**MVC模块(Spring WebMVC)**：Spring的MVC是一个全功能的构建Web应用程序的MVC的实现。通过策略接口,MVC框架变成为高度可配置的。MVC容纳了大量视图技术。模型来由JavaBean来构成，存放与Map当中。而视图是一个接口，负责实现模型。控制器是一个逻辑代码，是Control的实现。



## 2 IOC控制反转与DI依赖注入

**IOC（Inversion of Control，控制反转）是spring的核心，贯穿始终。**
所谓IOC，对于spring框架来说，就是由spring来负责控制对象的生命周期和对象间的关系：
传统开发模式：对象之间互相依赖
IOC开发模式：IOC容器安排对象之间的依赖

Spring所倡导的开发方式就是：所有的类都会在Spring容器当中登记，告诉Spring你是一个什么东西，你需要什么东西。
然后Spring会在系统运行到适当的时候把你所需要的东西主动送给你。
同时，也把你交给其他需要你的东西。
所有类的创建、销毁都由Spring控制。
也就是说：控制对象生存周期的不在是引用他的对象而是Spring。
对于某个具体的对象而言：以前是他控制其他对象，现在是**所有的对象都被Spring所控制**。
所以这就叫控制反转。



## 3 AOP-面向切面切面的编程技术

### AOP的基本概念：

​	在软件行业里，AOP为Aspect Oriented Programming的缩写，意为：**面向切面编程**，通过**预编译方式**和**运行期动态代理**实现程序功能的统一维护的一种技术。
​	AOP将应用系统分为两个部分：核心业务逻辑以及**横向的通用逻辑**，也就是所谓的面。
​	例如：所有大中型都要涉及到的持久化的管理、事物管理、安全管理、日志管理以及调试管理等等。

​	在Spring当中，提供了面向切面编程丰富的支持，允许通过分离应用的业务逻辑与系统级的服务进行内聚性的开发。应用对象只实现他们应该做的，也就是完成业务逻辑，仅此而已。他们并不负责，甚至是不会意识到其他的系统级别的关注点，例如：日志和事物支持等等。

### AOP与OOP的关系？

​	在软件行业当中，AOP是对OOP面向对象编程的一种有益的补充。同时，AOP也是OOP的延续，是软件开发中的一个热点，也是Spring框架当中一个非常重要的内容。我们可以这么理解：
​	面向对象编程OOP是从静态角度考虑程序结构，即OOP对业务处理过程中的实体以及属性和行为进行了抽象的封装，以获得更加清晰，高效果的国际化氛围，研究的是一种静态的领域。
​	面向切面的编程AOP是从动态角度考虑程序运行过程，即是针对业务处理过程中的切面进行提取，他所面对的是处理过程中的某个步骤或者阶段，研究的是一种动态的领域。

### 那么AOP的主要功能是什么呢？

​	主要是用于系统级别的功能，例如：日志记录、性能统计、安全控制、事物处理、异常处理等等这些主要功能。

### 那么接下来介绍下AOP的主要意图：

​	AOP主要是将日志记录，性能统计，安全控制，事物处理，异常处理等代码从业务逻辑代码中划分出来。通过对这些行为的分离，我们希望可以将他们独立到非指导性业务逻辑方法当中。继而改变这些行为的时候，不影响业务逻辑代码的处理。
​	也就是说：AOP把一些常用的服务进行模块化，并且使用声明的方式将这些组件使用到其他的业务组件当中去。这样做的结果就是每一个业务组件只需要关心自己的业务逻辑，而不要去了解一些常用的服务组件，这样就保证了更高的内聚性。

### AOP的存在价值：

​	AOP 专门用于处理系统中分布于各个模块中的交叉关注点的问题，在 Java EE 应用中，常常通过 AOP 来处理一些具有横切性质的系统级服务，如**事务管理、安全检查、缓存、对象池管理**等，AOP 已经成为一种非常常用的解决方案

### 那么我们为什么要用到AOP呢？

​	是因为由于系统会有很多不同的组件,每一个组件负责一块特定的功能。然而呢，我们希望每一个组件只关心他自身的核心功能，但是在系统中会有一些组件比如日志模块，事物管理模块和安全模块等等这些组件会比较频繁的融入到其他核心业务逻辑组件当中去。
​	这些常用的组件会分散到其他多个组件当中，这样会带来的麻烦是：
1.如果这些常用的组件经常发生变化，那么我们需要在多个其他相应组件当中进行修改。
2.这样使得我们的组件代码因为插入了与自身业务无关的核心组件而变的很混乱。



## 4 Java反射概念解析

首先先来看一下JAVA反射的概念：

### **JAVA反射（Reflection**）

​	在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。
Java的反射机制是java被称为**动态语言**的一个关键性质。
那么反射机制所能实现的功能有：

1.    只要给定类的全名，即可通过反射获取类的所有信息。
2.    反射可以在程序运行时获取任意一个对象所属的类对象。
3.    在运行时可以获取到类中所有属性对象，并对其操作（包括私有属性）。
4.    在运行时可以获取到类中、父类中所有方法，并调用。
  目前主流的应用框架如Struts2、Hibernate、Spring、SpringMVC等框架的核心全部是利用Java的反射机制来实现的。
  在学习JAVA的反射机制前我们得先了解下JAVA中的几大对象 Class、Constructor、Field、Method
  首先先来看下JAVA中的Class对象
  Class其实就是类的类型，比如字符串类型就是String，整形类型就是Integer，String和Integer类型就是Class。
  Class对象是运行时获取的

### Class对象的常用方法介绍

​	getName()    获得类中完整名称
​	getDeclaredFields()    获得类中的所有属性
​	getDeclaredMethods()     获得类中所有的方法
​	getConstructors()     获得类构造方法
​	newInstance()     实例化类对象
注：newInstance()方法为实例化空参数的类对象时使用。



### 反射相关的几个对象

接下来再来认识一下**Constructor对象**
​	Constructor：类的构造函数反射类，通过getConstructor方法可以获得类的所有构造函数反射对象数组。其中最主要的一个方法是newInstance，通过该方法可以创建一个对象类的实例。相当于new关键字

接下来再来认识一下**Field对象**
​	Field：用于表示类中、接口中属性对象的类。包名：java.lang.reflect.field利用他可以用来操作类中所有的属性和属性的信息，不论公有还是私有的。

Field类的常用方法介绍：
​	getName()    获得属性名称
​	getType()    获得属性类型
​	get(Object obj)     取得obj对象中这个属性的值
​	set(Object obj, Object value)     向obj对象中这个属性赋值value
​	setAccessible(true)     启用/禁用访问控制权限

我们先来介绍下Class对象中获取Field对象常用的两种方法：
​	Field[] getDeclaredFields()      获取类中所有的属性信息，包括私有属性信息
​	Field[] getFields()     获取类中所有的公共属性信息

最后再来看一下**Method对象**
​	Method：表示类中、接口中方法对象的类。包名：java.lang.reflect.method 利用他可以操作类中所有的方法，不论公有还是私有。	

Method对象的常用方法介绍：
​	getName()    获得方法名称
​	getReturnType()    获得方法返回值类型
​	**invoke(Object obj, Object... args)      利用obj对象调用该方法**
​	getParameterTypes()    获得方法所有参数类型，按照顺序返回Class数组
​	getDeclaredAnnotations()     获取方法的全部注解

我们先来看一下Class类中获取Method对象的几种常用方式：
​	Method[] getDeclaredMethods() 获取类中定义的所有方法。
​	Method getMethod(String name, Class<?>... parameterTypes)   获取某个特定的方法，第一个参数为方法名称，第二个参数为方法参数的类对象。当方法具有多个参数时，传入的是Class数组；当参数为0时，传入null。
在这里必须重点介绍invoke(Object obj, Object... args)  方法。
​	这个方法是Object对象执行方法的反射，由被执行的方法调用（反着调用，也算反射的小特征），传入的第一个参数是调用该方法的Object对象，第二个参数是调用该方法时应传入的参数。当具有多个参数时，传入Object数组；当参数为0时，传入空的Object数组。



BeanFactory的初始化顺序如下:

1.    首先创建相关的配置文件
2.    Spring通过BeanFactory来装载配置文件
3.    启动IoC容器
4.    获取Bean实例

其中：通过BeanFactory启动IoC实例时并不会初始化配置文件当中定义的Bean，初始化的操作发生在第一个调用的时候。对于单实例的Bean来说BeanFactory会缓存Bean的实例，所以第二次使用getBean的方法时就可以直接从IoC容器的缓存中来获取Bean的实例了。

另外，在初始化BeanFactory时必须为其提供一种日志框架，我们使用的是Log4J，即在类路径下提供Log4J的配置文件，这样启动Spring容器时才不会报错。



## 5 Bean生命周期介绍

### 5.1 IoC容器的初始化流程分析

​	IOC容器的初始化分为三个过程实现：

#### 1. Resource资源定位

​	这个Resouce指的是**BeanDefinition**的资源定位。这个过程就是容器找数据的过程，就像水桶装水需要先找到水一样。

#### **2. BeanDefinition的载入过程**

​	这个载入过程是把用户定义好的Bean表示成Ioc容器内部的数据结构，而这个容器内部的数据结构就是BeanDefition。

​	这个过程是最繁琐，也是最重要的一个过程。这一个过程分为以下几步，

```
	1. 构造一个BeanFactory,也就是IOC容器
	2. 调用XML解析器得到document对象 
	3. 按照Spring的规则解析BeanDefinition
```

​	对于以上过程，都需要一个入口，也就是前面提到的**refresh()**方法，这个方法在AbstractApplicationContext类中，它描述了整个ApplicationContext的初始化过程，比如BeanFactory的更新，MessageSource和PostProcessor的注册等等。它更像是个初始化的提纲，这个过程为Bean的声明周期管理提供了条件。

##### 	**2.1 构建IOC容器**

​	这个过程的入口是refresh方法中的obtainFreshBeanFactory（）方法。整个过程构建了一个DefaultListableBeanFactory对象，这也就是IOC容器的实际类型。这一过程的核心如下：

​	2.1.1 obtainFreshBeanFactory

​		这个方法的作用是通知子类去初始化ioc容器，它调用了AbstractRefreshableApplicationContext的refreshBeanFactory 方法 进行后续工作。同时在日志是debug模式的时候，向日志输出初始化结果。

​	2.1.2 refreshBeanFactory

​		这个方法在创建IOC容器前，如果已经有容器存在，那么需要将已有的容器关闭和销毁，保证refresh之后使用的是新建立的容器。同时 在创建了空的IOC容器后，开始了对BeanDefitions的载入



##### 	2.2 解析XML文件

​	对于Spring，我们通常使用xml形式的配置文件定义Bean，在对BeanDefition载入之前，首先需要进行的就是XML文件的解析。整个过程的核心方法如下：

​	2.2.1 loadBeanDefinitions(DefaultListableBeanFactory beanFactory)

​		这里构造一个XmlBeanDefinitionReader对象，把解析工作交给他去实现

​	2.2.2 loadBeanDefinitions

​	（1） AbstractXmlApplicationContext类 ，利用reader的方法解析，向下调用（Load the bean definitions with the given XmlBeanDefinitionReader.）

​	（2） AbstractBeanDefinitionReader 类 解析Resource  向下调用

​	（3） XmlBeanDefinitionReader  

​	（4） doLoadBeanDefinitions

​	下面是它的核心方法，第一句调用Spring解析XML的方法得到document对象，而第二句则是载入BeanDefitions的入口

```java
try {
	Document doc = doLoadDocument(inputSource, resource);
	return registerBeanDefinitions(doc, resource);
}
```



##### 	2.3 解析Spring数据结构

​		这一步是将document对象解析成spring内部的bean结构，实际上是AbstractBeanDefinition对象。这个对象的解析结果放入BeanDefinitionHolder中，而整个过程是由BeanDefinitionParserDelegate完成。

​	2.3.1 registerBeanDefinitions

​		解析BeanDefinitions的入口，向下调用doRegisterBeanDefinitions方法

​	2.3.2 doRegisterBeanDefinitions

​		定义了BeanDefinitionParserDelegate 解析处理器对象，向下调用parseBeanDefinitions 方法

​	2.3.3 parseBeanDefinitions

​		从document对象的根节点开始，依据不同类型解析。具体调用parseDefaultElement和parseCustomElement两个方法进行解析。这个主要的区别是因为bean的命名空间可能不同，Spring的默认命名空间是“http://www.springframework.org/schema/beans”，如果不是这个命名空间中定义的bean，将使用parseCustomElement方法。

​	2.3.4 parseDefaultElement

​		这个方法就是根据bean的类型进行不同的方法解析。

​	2.3.5 processBeanDefinition

​		这个方法完成对普通，也是最常见的Bean的解析。这个方法实际上完成了解析和注册两个过程。这两个过程分别向下调用parseBeanDefinitionElement和registerBeanDefinition方法。

​	2.3.6 parseBeanDefinitionElement

​		定义在BeanDefinitionParserDelegate 类中，完成了BeanDefition解析工作。在这里可以看到，AbstractBeanDefinition实际上spring的内部保存的数据结构



#### 3. 注册BeanDefition

​	完成了上面的三步后，目前ApplicationContext中有两种类型的结构，一个是DefaultListableBeanFactory，它是Spring IOC容器，另一种是若干个BeanDefinitionHolder，这里面包含实际的Bean对象，AbstractBeanDefition。

​	需要把二者关联起来，这样Spring才能对Bean进行管理。在DefaultListableBeanFactory中定义了一个Map对象，保存所有的BeanDefition。这个注册的过程就是把前面解析得到的Bean放入这个Map的过程。

​	3.1 registerBeanDefinition

​		注册的入口，对于普通的Bean和Alias调用不同类型的注册方法进行注册。

​	3.2 registerBeanDefinition 

​		注册Bean 定义在DefaultListableBeanFactory中

​	3.3 registerAlias 

​		定义在SimpleAliasRegistry类，对别名进行注册

```java
@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
```

![img](https://img-blog.csdn.net/20150724202916051?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



​	Spring容器中的Bean拥有明确的生命周期，由多个特定的生命阶段组成，每个生命阶段都允许外界对Bean施加控制。在Spring中，我们从Bean的作用范围和实例化Bean时所经历的一系列阶段来描述Bean的生命周期：
接下来，我们从BeanFactory和ApplicationContext两个方面来分析Bean的生命周期：

### 5.2 BeanFactory中的Bean的生命周期：

![Bean生命周期](Bean生命周期.png)

​	Spring Bean的生命周期:  **实例化Bean ->设置对象属性（依赖注入）->处理Aware接口 ->BeanPostProcessor ->InitializingBean 与 init-method -> [postProcessAfterInitialization(Object obj, String s)] ->创建完成,可以使用   ->DisposableBean ->destroy-method**

![img](https://img-blog.csdn.net/20160528234908002?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

Bean的完整生命周期从Spring容器着手实例化Bean开始一直到最终销毁Bean，这当中经过了许多关键点，而每一个关键点都涉及到特定的方法调用。那么我们将这些方法大致划分为三类:

1.    Bean自身的方法，如调用Bean的构造函数实例化Bean，调用set方法来设置Bean的属性值以及通过配置文件当中init-method和destroy-method所指定的方法
2.    Bean级生命周期的接口方法，比如BeanNameAware、BeanFactoryAware、initializingBean以及DisposableBean这些接口方法都由Bean类来直接实现。
3.    容器级生命周期接口方法

### 5.3ApplicationContext中的Bean的生命周期

![img](https://img-blog.csdn.net/20160528234920802?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



### 5.4 BeanFactory和ApplicationContext

#### 1. **BeanFactory**

​	BeanFactory 是 Spring 的“心脏”。它就是 Spring IoC 容器的真面目。Spring 使用 BeanFactory 来**实例化、配置和管理 Bean。**

​	BeanFactory：是IOC容器的核心接口， 它定义了**IOC的基本功能**，我们看到它主要定义了**getBean**方法。getBean方法是IOC容器获取bean对象和引发依赖注入的起点。方法的功能是返回特定的名称的Bean。

​	BeanFactory 是初始化 Bean 和调用它们生命周期方法的“吃苦耐劳者”。注意，BeanFactory **只能管理单例**（Singleton）Bean 的生命周期。它不能管理原型(prototype,非单例)Bean 的生命周期。这是因为原型 Bean 实例被创建之后便被传给了客户端,容器失去了对它们的引用。



1. XmlBeanFactory通过Resource装载Spring配置信息并启动IoC容器，然后就可以通过factory.getBean从IoC容器中获取Bean了。


2. 通过BeanFactory启动IoC容器时，并不会初始化配置文件中定义的Bean，**初始化动作发生在第一个调用时**。
3. 对于单实例（singleton）的Bean来说，BeanFactory会**缓存Bean**实例，所以第二次使用getBean时直接从IoC容器缓存中获取Bean。



#### 2. **ApplicationContext**

​	如果说BeanFactory是Spring的心脏，那么ApplicationContext就是完整的躯体了，ApplicationContext由BeanFactory派生而来，提供了更多**面向实际应用**的功能。在BeanFactory中，很多功能需要以编程的方式实现，而在ApplicationContext中则可以通过配置实现。

​	BeanFactorty接口提供了配置框架及基本功能，但是无法支持spring的aop功能和web应用。而ApplicationContext接口作为BeanFactory的派生，因而提供BeanFactory所有的功能。而且ApplicationContext还在功能上做了**扩展**，相较于BeanFactorty，ApplicationContext还提供了以下的功能： 

​	（1）MessageSource, 提供**国际化**的消息访问  
​	（2）**资源访问**，如URL和文件  
​	（3）事件传播特性，即支持**aop**特性
​	（4）载入**多个（有继承关系）上下文** ，使得每一个上下文都专注于一个特定的层次，比如应用的web层 

​	ApplicationContext：是IOC容器另一个重要接口， 它继承了BeanFactory的基本功能， 同时也继承了容器的高级功能，如：MessageSource（国际化资源接口）、ResourceLoader（资源加载接口）、ApplicationEventPublisher（应用事件发布接口）等。



#### 3. **二者区别**

##### 	**1. 加载时机**

​	BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化，这样，我们就不能发现一些存在的Spring的配置问题。

​	而ApplicationContext则相反，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误。 相对于基本的BeanFactory，ApplicationContext 唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。

​	BeanFacotry延迟加载,如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常；而ApplicationContext则在初始化自身时检验，这样有利于检查所依赖属性是否注入；所以通常情况下我们选择使用 ApplicationContext。
​	应用上下文则会在上下文启动后**预载入所有的单实例Bean**。通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

##### 	**2. 后置处理器注册方式**

​	BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：

​	**BeanFactory需要手动注册，而ApplicationContext则是自动注册。**（Applicationcontext比 beanFactory 加入了一些更好使用的功能。而且 beanFactory 的许多功能需要通过编程实现而 Applicationcontext 可以通过配置实现。比如后处理 bean ， Applicationcontext 直接配置在配置文件即可而 beanFactory 这要在代码中显示的写出来才可以被容器识别。 ）

##### 	**3. 目标使用者**

​	beanFactory主要是面对与 spring 框架的基础设施，面对 spring 自己。而 Applicationcontex 主要面对与 spring 使用的开发者。基本都会使用 Applicationcontex 并非 beanFactory 。

#### 	**4.总结**

作用：

1. BeanFactory负责读取bean配置文档，管理bean的加载，实例化，维护bean之间的依赖关系，负责bean的声明周期。

2. ApplicationContext除了提供上述BeanFactory所能提供的功能之外，还提供了更完整的框架功能：

   a. 国际化支持
   b. 资源访问：Resource rs = ctx. getResource(“classpath:config.properties”), “file:c:/config.properties”
   c. 事件传递：通过实现ApplicationContextAware接口

3. 常用的获取ApplicationContext

   FileSystemXmlApplicationContext：从文件系统或者url指定的xml配置文件创建，参数为配置文件名或文件名数组，有相对路径与绝对路径。

```java
ApplicationContext factory=new FileSystemXmlApplicationContext("src/applicationContext.xml");
ApplicationContext factory=new FileSystemXmlApplicationContext("E:/Workspaces/MyEclipse 8.5/Hello/src/applicationContext.xml");
```

​	ClassPathXmlApplicationContext：从classpath的xml配置文件创建，可以从jar包中读取配置文件。			ClassPathXmlApplicationContext 编译路径总有三种方式：

```java
ApplicationContext factory = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
ApplicationContext factory = new ClassPathXmlApplicationContext("applicationContext.xml"); 
ApplicationContext factory = new ClassPathXmlApplicationContext("file:E:/Workspaces/MyEclipse 8.5/Hello/src/applicationContext.xml");
```

​	XmlWebApplicationContext：从web应用的根目录读取配置文件，需要先在web.xml中配置，可以配置监听器或者servlet来实现

```xml
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

或

```xml
<servlet>
	<servlet-name>context</servlet-name>
	<servlet-class>org.springframework.web.context.ContextLoaderServlet</servlet-class>
	<load-on-startup>1</load-on-startup>
</servlet>
```

这两种方式都默认配置文件为web-inf/applicationContext.xml，也可使用context-param指定配置文件

```xml
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>/WEB-INF/myApplicationContext.xml</param-value>
</context-param>
```

## 6 事务

### 6.1 事务

事务是指 用户在执行其定义的一个数据库操作序列时，操作序列作为一个不可分割的工作单位，这些操作的执行要么全部都生效要么全部都不生效。 
事务的开始和结束可以由用户显式控制。如果用户没有显式定义事务，则按照数据库缺省规定自动划分事务。

在SQL中，事务相关的语句有3条：

BEGIN TRANSACTION 
COMMIT 
ROLLBACK
1
2
3
BEGIN TRANSACTION 表示事务开始，一个事务开始后，必须以COMMIT或ROLLBACK结束。
COMMIT 表示提交，即提交事务的所有操作。具体的说就是将事务中所有对数据库的更新写回到磁盘上的物理数据库中去，事务正常结束。
ROLLBACK 表示回滚，即在事务运行的过程中发生了某种故障，事务不能继续执行，系统将事务中对数据库的所有已完成的操作全部撤销，回滚到事务开始时的状态。

#### 事务的特性ACID

​	事务具有四个特性：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）和持续性（Durability）。这四个特性简称为ACID特性（ACID properties）。

**原子性** 
​	事务是数据库的逻辑工作单位，事务中包括的所有操作要么都做，要么都不做。

**一致性** 
​	事务执行的结果必须是使数据库从一个一致性状态变成另一个一致性状态。因此当数据库指包含成功事务提供的结果时，就说数据库处于一致性状态。如果数据库系统运行中发生故障，有些事务尚未完成就被迫中断，这些未完成事务对数据库所做的修改有一部分一写入物理数据库，这时数据库就处于一种不正确的状态，或者说是不一致的状态。例如，在银行中有A、B两个账号，现在公司想从账号A中取出一万元，存入账号B中。那么就可以定义一个事务，改事务包括两个操作，第一个操作是从账号A中减去一万元，第二个操作是向账号B中加入一万元。这两个操作要么全做，要么全不做。全做或者全不做，数据库都处于一致性状态。如果只做一个操作则用户逻辑上就会发生错误，少了一万元，这时数据库就出现不一致性状态。可见一致性与原子性密切相关。

**隔离性** 
​	一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对其他并发事务是隔离的，并发执行的各个事务之间不能相互干扰。

**持续性** 
​	持续性也称永久性（Permanence），指一个事务一旦提交，它对数据库中的数据的改变就应该是永久性的。接下来的其他操作或故障不应该对其执行结果有任何影响。

​	事务是恢复和并发控制的基本单位。保证事务ACID特性是事务管理的重要任务。事务ACID特性可能遭到破坏的因素有：

​	(1) 多个事务并行运行时，不同事务的操作交叉执行； 
​	(2) 事务在运行过程中被强行终止。

​	在第一种情况下，数据库管理系统必须保证多个事务的交叉运行不影响这些事务的原子性。在第二种情况下，数据库管理系统必须保证被强行终止的事务对数据库和其他事务没有任何影响。 
这些就是数据库管理系统中恢复机制和并发控制机制的责任。

#### 事务隔离级别

​	在许多事务处理同一个数据时，特别是并发处理数据时，如果没有采取有效的隔离机制，就会带来一些问题。比如出现脏读、不可重复读、幻读等异常情况。

​	**脏读**：一个事务读取到另一个事务未提交的更新数据。
​	**不可重复读**：一个事务两次读取同一行的数据，结果得到不同状态的结果，中间正好另一个事务更新或者删除了该数据，两次结果相异，不可被信任。
​	**幻读**(也叫虚读)：一个事务执行两次查询，第二次结果集包含第一次中没有 的数据行，造成两次结果不一致，这是另一个事务在这两次查询中间插入了数据造成的。

```
非重复度和幻像读的区别

	非重复读是指同一查询在同一事务中多次进行，由于其他提交事务所做的修改或删除，每次返回不同的结果集，此时发生非重复读。(A transaction rereads data it has previously read and finds that another committed transaction has modified or deleted the data. )

	幻像读是指同一查询在同一事务中多次进行，由于其他提交事务所做的插入操作，每次返回不同的结果集，此时发生幻像读。(A transaction reexecutes a query returning a set of rows that satisfies a search condition and finds that another committed transaction has inserted additional rows that satisfy the condition. )
	
	如果使用锁机制来实现这两种隔离级别，在可重复读中，该sql第一次读取到数据后，就将这些数据加锁，其它事务无法修改这些数据，就可以实现可重复读了。但这种方法却无法锁住insert的数据，所以当事务A先前读取了数据，或者修改了全部数据，事务B还是可以insert数据提交，这时事务A就会 发现莫名其妙多了一条之前没有的数据，这就是幻读，不能通过行锁来避免。需要Serializable隔离级别，读用读锁，写用写锁，读锁和写锁互斥，这么做可以有效的避免幻读、不可重复读、脏读等问题，但会极大的降低数据库的并发能力。
```

​	表面上看，区别就在于非重复读能看见其他事务提交的修改和删除，而幻像能看见其他事务提交的插入。

​	由于对事务的并发控制需要耗费性能，越准确的控制往往意味着耗费越多的性能。为了实现效率的最大化，通常在一些业务情况下我们允许一些异常情况发生（一般不影响最终结果）。根据异常发生的可能性和效率的差异，我们将事务控制划分为不同的级别，即事务的隔离级别。

事务的隔离级别定义了不同级别下可能发生的一些异常情况。

| 隔离级别                   | 脏读   | 不可重复读 | 幻读   |
| ---------------------- | ---- | ----- | ---- |
| 未提交读（Read uncommitted） | 可能   | 可能    | 可能   |
| 已提交读（Read committed）   | 不可能  | 可能    | 可能   |
| 可重复读（Repeatable read）  | 不可能  | 不可能   | 可能   |
| 可串行化（Serializable ）    | 不可能  | 不可能   | 不可能  |

**未提交读(Read Uncommitted)**：允许脏读，也就是可能读取到其他会话中未提交事务修改的数据

**提交读(Read Committed)**：只能读取到已经提交的数据。**Oracle,SQL Server**等多数数据库默认都是该级别 (不重复读)

**可重复读(Repeated Read)**：可重复读。Mysql默认隔离级别为可重复读,在同一个事务内的查询都是事务开始时刻一致的，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。在SQL标准中，该隔离级别消除了不可重复读，但是还存在幻象读。 

​	**Mysql默认隔离级别为可重复读。**其中，InnoDB和Falcon存储引擎通过**多版本并发控制**(MVCC，Multiversion Concurrency Control)机制**解决了幻象读**问题。

**串行化(Serializable)**：完全串行化的读，每次读都需要获得表级共享锁，读写相互都会阻塞。在这个级别，可能导致大量的超时现象和锁竞争



### 6.2 Spring事务

​	Spring对事务提供了两种支持方式：**编程式事务** 和 **声明式事务**。编程式事务是指通过在编程时直接在业务逻辑代码中写入事务控制的相关代码，与业务逻辑代码耦合，一般不建议使用这种方式；声明式事务则通过AOP的方式来进行事务控制，对事务的控制逻辑不会侵入到业务逻辑代码中。

#### Spring 编程式事务

使用方式

```java
public void transactionTest() {
    Connection conn = null;  
    UserTransaction tx = null;  
    try {  
        tx = getUserTransaction();                       //1.获取事务  
        tx.begin();                                    //2.开启JTA事务  
        conn = getDataSource().getConnection();           //3.获取JDBC  
        String sql = "select * from table";           //4.声明SQL  
        PreparedStatement pstmt = conn.prepareStatement(sql);//5.预编译SQL  
        ResultSet rs = pstmt.executeQuery();               //6.执行SQL  
        process(rs);                                   //7.处理结果集  
        closeResultSet(rs);                             //8.释放结果集  
        tx.commit();                                  //9.提交事务  
    } catch (Exception e) {  
        tx.rollback();                                 //10.回滚事务  
        throw e;  
    } finally {  
       close(conn);                                //11.关闭连接  
    }  
}
```

使用template

```java
//Spring 编程式事务
public void transactionTemplateTest() {
  TransactionTemplate transactionTemplate = new TransactionTemplate(txManager); 
  transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED); //设置事务隔离级别
  transactionTemplate.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);//设置为required传播方式
  ...
  transactionTemplate.execute(new TransactionCallbackWithoutResult() { 
      @Override 
      protected void doInTransactionWithoutResult(TransactionStatus status) {  
        //事务控制逻辑
        jdbcTemplate.update(SQL, args); 
        ...
  }}); 
}
```

#### Spring 声明式事务

声明式事务支持两种使用方式： 

1. ##### 在配置文件（xml）中声明相关事务规则

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       default-autowire="byName"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">

           <!-- 引入db.properties文件 -->
           <context:property-placeholder location="classpath:db.properties"/>

           <!-- 配置连接池（数据源） -->
           <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
               <property name="driverClass" value="${driverClass}"></property>
               <property name="jdbcUrl" value="${jdbcUrl}"></property>
               <property name="user" value="${user}"></property>
               <property name="password" value="${password}"></property>
               <property name="minPoolSize" value="${minPoolSize}"></property>
               <property name="maxPoolSize" value="${maxPoolSize}"></property>
               <property name="initialPoolSize" value="${initialPoolSize}"></property>
           </bean>

           <!-- 配置JdbcTemplate -->
           <bean class="org.springframework.jdbc.core.JdbcTemplate" id="j">
               <property name="dataSource" ref="dataSource"></property>
           </bean>

           <!-- 配置事务管理器，事务核心管理器,封装了所有事务操作。 关联连接池 -->
           <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
               <property name="dataSource" ref="dataSource"></property>
           </bean>

           <!-- 配置事务通知 -->
           <tx:advice id="txAdvice" transaction-manager="transactionManager">
               <tx:attributes>
                   <!-- 配置方法的事务属性，支持通配符 isolation:隔离级别 propagation:传播行为 read-only:是否只读 -->
                   <tx:method name="save*" isolation="REPEATABLE_READ"
                       propagation="REQUIRED" read-only="false" />
                   <tx:method name="persist*" isolation="REPEATABLE_READ"
                       propagation="REQUIRED" read-only="false" />
                   <tx:method name="update*" isolation="REPEATABLE_READ"
                       propagation="REQUIRED" read-only="false" />
                   <tx:method name="modify*" isolation="REPEATABLE_READ"
                       propagation="REQUIRED" read-only="false" />
                   <tx:method name="delete*" isolation="REPEATABLE_READ"
                       propagation="REQUIRED" read-only="false" />
                   <tx:method name="remove*" isolation="REPEATABLE_READ"
                       propagation="REQUIRED" read-only="false" />
                   <tx:method name="get*" isolation="REPEATABLE_READ"
                       propagation="REQUIRED" read-only="true" />
                   <tx:method name="find*" isolation="REPEATABLE_READ"
                       propagation="REQUIRED" read-only="true" />
                   <tx:method name="transfer" isolation="REPEATABLE_READ"
                       propagation="REQUIRED" read-only="false" />
               </tx:attributes>
           </tx:advice>

           <!-- 配合事务切入点 -->
           <aop:config>
               <!-- 配置切点表达式 -->
               <aop:pointcut expression="execution(* com.test.service.imp.*.*(..))" id="pointcut"/>
               <!-- 配置切面 : 通知+切点 advice-ref:通知的名称 pointcut-ref:切点的名称 -->
               <aop:advisor advice-ref="advice" pointcut-ref="pointcut"/>
           </aop:config>

           <!--开启注解的方式-->
           <tx:annotation-driven transaction-manager="transactioManager" />
   </beans>
   ```

##### 2.使用@Transactional 注解声明事务规则

```java
@Transactional
public void transactionTest() {
    //事务内逻辑
    jdbcTemplate.update(SQL, args); 
    ...
}
```

```
@Transactional可指定的事务属性

　　 @isolation：用于指定事务的隔离级别。默认为底层事务的隔离级别

　　 @noRollbackFor：指定遇到特定异常时不回滚事务

　　 @noRollbackForClassName：指定遇到特定的多个异常时不回滚事务

　　 @propagation：指定事务传播行为

　　 @readOnly：指定事务是否可读

　　 @rollbackFor：指定遇到特定异常时回滚事务

　　 @rollbackForClassName：指定遇到特定的多个异常时回滚事务

　　 @timeout：指定事务的超长时长。
```

#### Spring中的事务隔离级别

​	事务隔离级别：用来解决并发事务时出现的问题，其使用TransactionDefinition中的静态变量来指定。Spring中支持五种事务隔离级别方式：

```
ISOLATION_DEFAULT：默认隔离级别，即使用底层数据库默认的隔离级别；
ISOLATION_READ_UNCOMMITTED：未提交读；
ISOLATION_READ_COMMITTED：提交读，一般情况下我们使用这个；
ISOLATION_REPEATABLE_READ：可重复读；
ISOLATION_SERIALIZABLE：序列化。
```

#### Spring事务的传播方式

事务分为**物理事务**和**逻辑事务**:

​	物理事务：就是底层数据库提供的事务支持，如JDBC或JTA提供的事务；

​	逻辑事务：是Spring管理的事务，不同于物理事务，逻辑事务提供更丰富的控制，而且如果想得到Spring事务管理的好处，必须使用逻辑事务，因此在Spring中如果没特别强调一般就是逻辑事务。

Spring管理的事务是逻辑事务，物理事务和逻辑事务最大差别就在于事务传播行为，事务传播行为用于指定在多个事务方法间调用时，事务是如何在这些方法间传播的，**Spring共支持7种传播行为**：

```
PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。是Spring默认的传播行为。
PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价PROPAGATION_REQUIRED。嵌套事务使用数据库中的保存点来实现，即嵌套事务回滚不影响外部事务，但外部事务回滚将导致嵌套事务回滚。
```



#### Spring 事务实现原理--AOP 代理

​	在应用系统调用 事务声明 的目标方法时，Spring Framework 默认使用 **AOP 代理**，在代码运行时生成一个**代理对象**，根据事务配置信息，这个代理对象决定该声明事务 的目标方法是否由拦截器 TransactionInterceptor 来使用拦截，在 TransactionInterceptor 拦截时，会在目标方法开始执行之前创建并加入事务，并执行目标方法的逻辑, 最后根据执行情况是否出现异常，利用抽象事务管理器 AbstractPlatformTransactionManager 操作数据源 DataSource 提交或回滚事务。

​	Spring AOP 代理有 CglibAopProxy 和 JdkDynamicAopProxy 两种，以CglibAopProxy 为例，对于CglibAopProxy，需要调用其内部类的DynamicAdvisedInterceptor 的 intercept 方法。对于 JdkDynamicAopProxy，则调用其 invoke 方法。

#### Spring事务中需要注意的问题

**事务失效**问题--嵌套调用

​	在同一个代理对象内部，事务方法之间的直接**嵌套调用**，普通方法和事务方法之间的直接嵌套调用，都会造成事务异常！具体表现为某些传播行为不生效或者直接事务控制不生效。

```
@Service
public class DemoService {  

    @Transaction
    public void transactionMethod1()  
    {  
        op1();
        op2();
        ...
    }  
    public void commonMethod()  
    {  
        ...
        transactionMethod1();  //照顾基础不牢的朋友，这里相当于 this.transactionMethod1(); 
        ...  
    } 
    @Transaction
    public void transactionMethod2()  
    {  
        ...
        this.transactionMethod3();
        ...
    }  
    @Transaction(propagation= Propagation.REQUIRES_NEW)
    public void transactionMethod3()  
    {  
        op3();
        ...
    }  
} 
```

​	上面代码中，如果调用 DemoService 的 bean 对象的commonMethod ，则transactionMethod1里定义的事务将不生效（比如op2发生错误时，并不会回滚op1的操作），bean 调用 transactionMethod2时，transactionMethod2时里面调用的transactionMethod3也不会开启新的事务。

为什么会这样？

​	上面的实现机制中讲到，AOP的实现都是通过动态代理来实现，而AOP限制了我们只能在目标方法的开始和结束作为切点做切入处理增强。当动态代理对象最终调用的原始对象的目标方法时，并**不能干预到目标方法内的方法调用行为**，如果原始对象直接调用（用this.xxx方式）其他同类方法时，实际调用的是原始对象自身的方法，而不是代理类对象增强后（增加事务控制后）的方法。此时Spring对方法事务的控制（包括事务的传播行为、事务的隔离级别等）完全失效。



如何解决?

​	要想解决此类问题，主要都在于原始对象在调用对象内其他方法时，不要使用this.xxx的方式直接调用，通过**注入**或者**获取代理对象**的方式，**使用代理对象调用需要调用的方法**。下面列举几个解决方式：

- 1.注入自身，使用代理对象调用

```
@Service
public class DemoService {  

    @Autowired
    DemoService demoService;

    @Transaction
    public void transactionMethod1()  
    {  
        op1();
        op2();
        ...
    }  
    public void commonMethod()  
    {  
        ...
        //this.transactionMethod1() -> demoService.transactionMethod1()  
        demoService.transactionMethod1();  
        ...  
    }  
    ...
} 
```

- 2.使用AopContext,获取当前代理对象

  ```java
  @Service
  public class DemoService {

      @Transaction
      public void transactionMethod1()  
      {  
          op1();
          op2();
          ...
      }  
      public void commonMethod()  
      {  
          ...
          //this.transactionMethod1() -> ((DemoService)AopContext.currentProxy()).transactionMethod1();) 
          ((DemoService)AopContext.currentProxy()).transactionMethod1();
          ...  
      }  
      ...
  } 
  ```

  - 3.使用BeanFactory获取代理对象（代码略）




## 7 SpringMVC运行流程

### 	1.  所有请求由前端控制器(DispatcherServlet)接收,调用doDispatch()进行处理;

### 	2.  根据HandlerMapping中保存的请求映射信息,找到处理当前请求的处理器执行链( 包含目标方法和拦截器);

### 	3.  根据当前处理器找到对应的HandlerAdapter(适配器);

### 	4.  执行拦截器的preHandler()方法;

### 	5.  适配器执行目标方法,返回ModelAndView;

​		1) ModelAndView注解的方法提前运行;

​		2) 执行目标方法的时候( 确定目标方法的参数)

​			1) 有注解;

​			2) 没注解:

​				1) 看是否是Model,Map以及其他的;

​				2) 如果是自定义类型:

​					1) 如果隐含模型中有该参数,则从隐含模型中获取;

​					2) 如果没有,再看是否是SessionAttribute标注的属性,如果是则从Session中获取,如果Session中不包含该属性,则抛出异常;

​					3) 都不是以上情况,则利用反射创建对象并赋值;

### 	6.  执行拦截器的postHandler()方法;

### 	7.  处理结果(渲染视图)

​		**1) 如果有异常则使用异常解析器处理异常,处理完之后同样返回ModelAndView;**

​		**2) 调用render()进行页面渲染;**

​			1) 视图解析器根据视图名得到视图对象;

​			2) 视图对象调用render()方法进行渲染;

​		**3) 执行拦截器的afterCompletion()方法(逆序);**

