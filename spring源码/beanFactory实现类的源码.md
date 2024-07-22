### BeanFactory 实现与继承关系

![请添加图片描述](https://i-blog.csdnimg.cn/direct/3781596c176f417abe48db8a0f1fbd81.png)

### ListableBeanFactory

由bean工厂实现的BeanFactory接口的扩展，这些bean工厂可以枚举它们所有的bean实例，而不是按客户端请求逐个按名称进行bean查找。

### HierarchicalBeanFactory

由bean工厂实现的子接口，可以是层次结构的一部分。

父子级联 IoC 容器的接口，子容器可以通过接口方法访问父容器； 通过 HierarchicalBeanFactory 接口，
Spring 的 IoC 容器可以建立父子层级关联的容器体系，子容器可以访问父容器中的 Bean，但父容器不能访问子容器的 Bean。

### ConfigurableBeanFactory

由大多数bean工厂实现的配置接口。
这个扩展的接口只是为了允许框架内部的即插即用和对bean工厂配置方法的特殊访问。

ConfigurableBeanFactory继承了  HierarchicalBeanFactory 和  SingletonBeanRegistry。

```
public interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry {

}
```

### AutowireCapableBeanFactory

BeanFactory的扩展接口，实现该接口能够自动装配。



### SingletonBeanRegistry

定义了共享bean实例的注册中心。


```

	/**
	 * 在bean注册中心通过bean名称将提供的对象注册为单例对象。提供的对象需要被完全初始化。
	 */
	void registerSingleton(String beanName, Object singletonObject);

	/**
	 * 返回以给定名称注册的(原始)单例对象
	 */
	@Nullable
	Object getSingleton(String beanName);

	/**
	 *  检查此注册表是否包含具有给定名称的单例实例。
	 * 
	 */
	boolean containsSingleton(String beanName);

	/**
	 * 返回在此注册中心中注册的单例bean的名称。
	 * 
	 */
	String[] getSingletonNames();

	/**
	 * 返回在此注册中心中注册的单例bean的数量。
	 */
	int getSingletonCount();

	/**
	 * 返回使用的单例互斥锁.
	 * 
	 */
	Object getSingletonMutex();

}

```