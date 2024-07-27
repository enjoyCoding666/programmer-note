### IoC (Inverse of Control)
IoC ，也就是 控制反转。

对于软件来说，即某一接口具体实现类的选择控制权从调用类中移除，转交给第三方决定，即由Spring容器借由Bean配置来进行控制。

Martin Fowler提出了DI(Dependency Injection,依赖注入)的概念用来代替IoC, 即让调用类对某一接口实现类的依赖关系由第三方（容器或协作类）注入，以移除调用类对某一接口实现类的依赖。


### loC的类型
Spring的 IoC 支持构造函数注入和属性注入。

Spring通过一个配置文件描述Bean及Bean之间的依赖关系，利用Java语言的反射功能实例化Bean并建立Bean之间的依赖关系。
Spring的IoC容器在完成这些底层工作的基础上，还提供了Bean实例缓存、生命周期管理、Bean实例代理、事件发布、资源装载等高级服务。

### BeanFactory

Bean工厂(com.springframework.beans..factory..BeanFactory)是Spring框架最核心的接口，它提供了高级IoC的配置机制。BeanFactory使管理不同类型的Java对象成为可能。一般称BeanFactory为IoC容器。

而BeanFactory是类的工厂，它可以创建并管理各种类的对象。
Spring 称这些被创建和管理的Java对象为Bean。


### ApplicationContext
应用上下文(com.springframework.context..ApplicationContext)建立在BeanFactory基础之上，提供了更多面向应用的功能，易于创建实际应用。我们称ApplicationContext为应用上下文。


### BeanFactory和ApplicationContext 的区别
* BeanFactory 是Spring框架的基础设施，面向Spring本身；
  ApplicationContext 面向使用Spring框架的开发者，开发中可以直接使用 ApplicationContext。

* ApplicationContext是BeanFactory的子接口，它提供了更多的功能和扩展，如国际化支持、事件发布、AOP集成、资源加载、消息解析等。

* ApplicationContext的初始化和 BeanFactory有一个重大的区别：BeanFactory在初始化容器时，并未实例化Bean,直到第一次访问某个Bean时才实例化目标Bean;而ApplicationContext则在初始化应用上下文时就实例化所有单实例的Bean。

### 参考资料
《精通Spring+4.x++企业应用开发实战》
