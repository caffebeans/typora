#                               spring知识点       



 



~~~ 
总：当前问题回答的是哪些具体的点
分：以详细的方式细节描述相关的知识点，如果有哪些不清楚，直接忽略过去
  突出一些技术名词（核心概念，接口，类，关键方法）
  避重就轻：没有重点
  一个问题占用面试官多长时间
~~~





## 1. 谈谈spring Ioc的理解，原理与实现

> 控制反转： 理论思想，原来的对象是由使用者进行控制，有了spring之后，可以把整个对象交给sping容器来帮我们进行管理



**总：** 

  DI: 信赖注入，把相应的属性值注入到具体的对象中，@autowired,popluateBean完成属性的注入

 容器： 存储对象，使用map结构来存储，在spring中一般存在三级缓存，singletonObjects存完整的bean对象，整个的生命同期从创建到销毁的过程全部都是由容器进来管理**（生命周期）**。

 

**分：**

1. 一般聊ioc容器的时候要涉及到容器的的创建过程（beanfactory,defaultLisenableBeanFactory，向bean工厂中设置一些参数（beanPostProcessor,aware）

2. 加载解析bean对象，准备在创建的bean对象的定义beanDefinition,(xml或者注解的解析过程)

3. beanFactoryPostProcessor的处理此处是扩展点

4. BeanpostPorcessor的注册功能，方便后续对bean对象完具体的扩展功能（**对占位符的处理**）

5. 通过反射的方式对BeanDefinition对象实例完成具体的bean对象。

6. bean对象的初始化过程（填充属性，调用aware子类的方法，调用BeanPostProcessor前置处理方法，调用init-method方法。

7. 生成完整的bean对象，通过getbean()方法可以直接获得。

8. 销毁过程

   面试官，这是我对ioc的整体理解，包含一些详细的处理过程，您看一下还有什么问题，可以指导我一下。

   具体的细节我记不太清楚了，但是spring中的bean都是通过反射方式生成的，同时其中包括很多的扩展点，比如最长用的是对BeanFacory的扩展，除此之外，ioc中最核心的也是对生命周期的管理。





## 2. ioc的底层实现

> 底层原理： 工作原理，过程，数据结构，流程，设计模式，设计思想
>
> 你对它的理解和你了解的实现过程
>
> 反射，工厂，设计saaa（会的说，不会的不说），关键的几个方法
>
> createBeanFactory,getBean,doGetBean,createBean,doCreateBean,popluateBean
>
> 加do的方法是实际上干活的方法



1. 通过createBeanFactory创建一个Bean工厂（DefaultListableBeanFactory）

2. 开始创建对象，因为容器中的对象默认都是单例的，所以通优先通过getbean,doGetBean从容器中查找，如果找不到，

3. 通过createBean,doCreateBean方法，以反射的方法创建对象，一般使用无参的构造方法（getClearConstrct,newInstance)

4. 进行对象的属性填充populateBean

5. 进行其它初始化操作（initalizingBean)

   



## 3. bean的生命周期



> 记住图中的流程
>
> 在表述的时候不要只说图中的关键技术点，要学会扩展描述



![1](https://img-blog.csdn.net/20180707170607708?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h5eGhiajE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

1. 实例化bean: 以反射方式生成对象

2. 填充bean的属性：pooluateBean(),循环依赖的问题（三级缓存的问题）

3. 调用aware接口的相关方法：

4. 调用BeanPostPorcessor中的前置处理方法：使用比较多的有ApplicationProcesor

   在这一系列中，会设置env

5. 调用initmethod方法：invokeInitMethod(),判断是否实现了imitlatizingBean接口
6. 调用BeanPostProcessor的后置处理方法，spring的aop就是在这个地方实现的。AbstractautoProxycreate
7. 获取到完整的对象，可以通过getBean的方式来进行对象的获取。

8. 销毁流程： 1 判断是否实现了dispoableBean接口， 2， 调用destoryMethody方法。



## 4. 循环依赖问题



> 三级缓存+提前暴露对象+aop

<img src="http://09itblog.site/wp-content/uploads/2020/06/bean-create-2-1.jpg" alt="h" style="zoom:100%;" />



###### **形成闭环的原因**

总：什么是循环依赖, A依赖B， B 依赖A

分：先说bean的创建过程，实例化，初始化（属性填充）

1. 先创建A对象，实例化A对象，此时A对象的属性为空，填充B属性
2. 从容器中查找B对象，如果能够找到B对象，则对A直接进行赋值操作，如果找不到
3. 实例化B对象，此时B对象的A 属性为空，填充属性a
4. 从容器中查找A对象，找不到，直接创建



~~~ 
   此时，如果仔细琢磨的话，会发现对象A是存在的，只是A对不是一个完整的状态。只是完成了实例化，但是没在完成初始化，如果在程序调用过程中，拥有了某个对象的引用，能否在后期给他完成赋值操作，可以优先把非完整状态的对象优先赋值，等待后续操作来完成赋值，相当于提前暴露了某个不完整对象的引用，所以解决循环依赖的核心在于实例化与初始化分开操作，这也是解决循环依赖的关键。
~~~



~~~
   当所有的对象都完成实例化与初始化之后，还要把完整的对象入到容器中，此时容器中的对象存在几个状态（完成实例化但未未完成初始化，完整状态），因为对象都在容器中，所以要使用不同的map结构来进行存储，此时也就有了一级缓存与二级缓存。如果一级缓存中已经存在了该对象，那么二级缓存中就不会存在相同的同名对象，因为他们 的查找顺序是1，2这样来查找的。
~~~



**为什么要三级缓存**

三级缓存放value放得是object类型，是一个函数式接口，存在的意义是保证在容器中只存在一个同名的bean对象。

如果一个对象需要被代理，或者说需要生成代理对象，要不要优先生成一个普通对象。

普通对象和代理对象是不能够同时出现在容器中的，因为当一个对象需要被代理时，就需要使用代理对象来覆盖掉之前的普通对象。





## 5. Bean Factory与FacoryBean有什么区别



**相同点：** 都是用来创建bean对象的

**不同点：**使用beanFactory来创建对象的时候，必须要遵循严格的生命周期流程，太复杂了，如果想要简单的自定义某个对象的创建，同时创建完成后交给spring容器来管理，那么必须就在实现FactoryBean接口了。

 isSingleon: 是否是单例对象 

getObject: 自定义创建对象的过程（new,反射，动态代理）



## 6. spring中用到的设计模式



 单例模式： bean都是单例的

原型模式：指定作用域为prototype

工厂模式：BeanFactroy

模板方法： PostProcessorBeanFactroy,onRefresh

策略模式： xmlBeanDefintionReader

观察者模式： listener,event,multicast

适配器模式：Adapter

装饰者模式：Beanwrapper

责任链模式：使用aop的时候生成一个拦截器链

代理模式：动态代理

委托者模式：delegate





## 7. spring的AOP的底层实现



动态代理

aop是ioc的一个动态扩展，先有ioc，再有aop，只是在ioc的整个流程中新增的一个扩展点而已；Beanpostcessor

总： aop概念，应用场景，动态代理

分：bean的创建过程中有一个步骤可以对bean进行扩展实现，aop本身就是一个扩展功能，所以在BeanPostPorcessor的后置处理方法中来进行实现。

1. 代理对象的创建过程 
2. 通过jdk或者cglib的方式来生成代理对象
3. 在执行方法调用的时候，会调用到生成的字节码文件中，直接会找到DynamicAdvisortdInterceptor类中的些intercept方法，从此方法开始执行
4. 根据之前定义好的通知来生成拦截器链
5. 从拦截器链中依次获取每一个通知开始执行，在执行过程中，为了方便找到下一个通知是哪个，会有一个CblibMethodInvocation对象，找到的时候从-1的位置依次开始查找并且执行



## 8. Spring的事务是如何回滚的

> spring事务与aop相关

spring的事务是如何实现的？

总： Spring的事务是由aop实现的，首先生成具体的代理对象，然后按照aop的整套流程来执行的操作逻辑，正常情况下要通过通知来完成核心功能，但是事务不是通过通知来实现的，而是通过一个transavtionInterceptor来实现的，然后调用invoke来实现具体的逻辑

 分

1. 先做准备工作，解析各个方法上事务相关属性，根据具体的属性来判断是否开始新的事务

2. 当需要开启的时候，获取数据库连接，关闭自动提交功能，开起事务

3. 执行具体的sql逻辑操作

4. 在执行过程中，如果执行失败了，那么会通过completeTransactionAfterThrowing来完成事务的回滚操作，回滚的具体逻辑是通过doRollRack方法来实现的，实现提时候也要先获取连接对象，通过连接对象来回滚

5. 如果执行过程中，没有发生什么错误，那么就会通过commitTransactionAfterReturining来完成事务的提交操作，提交的具体逻辑是通过doCommit方法来实现的，实现的时候也要先获得连接，通过 连接对象来提交

6. 当事务执行完毕后需要清除相关事务的cleanupTranactionInfo

   如果想要聊得更加细致的话，需要知道transactioninfo,transactionstus.



## 9. 谈一下spring事务传播

> spring事务的传播特性大约有七种



Requried,Requires_new,nested,support,Not_Support,Never,Mandatory

**某一个事务嵌套另一个事务的时候怎么办？**

A方法调用B方法，AB方法都有事务，并且传播特性不同，那么A有异常，B怎么办，B有异常，A怎么办

**总：**事务的传播特性指的是不同方法的嵌套调用过程中，事务应该如何进行处理，是用同一个事务还是不同的事务，当出现异常的时候会回滚还是提交，两个方法之间的相关影响，在日常工作中，使用比较多的是required,requries_new,nested

**分** 1.  先说事务的不同分类，可以分为三种：支持当前事务，不支持当前事务，嵌套事务

   2. 如果外层是requires,内层方法是，required,required_new,nested

   3. 如果外层方法是required_new,内层方法是，required

   4. 如果外层方法是nested, 内层方法是，requried,required_new,nestd

      

      

      