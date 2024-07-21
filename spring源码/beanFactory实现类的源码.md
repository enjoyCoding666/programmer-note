### ListableBeanFactory
由bean工厂实现的BeanFactory接口的扩展，这些bean工厂可以枚举它们所有的bean实例，而不是按客户端请求逐个按名称进行bean查找。

### HierarchicalBeanFactory
由bean工厂实现的子接口，可以是层次结构的一部分。

父子级联 IoC 容器的接口，子容器可以通过接口方法访问父容器； 通过 HierarchicalBeanFactory 接口， 
Spring 的 IoC 容器可以建立父子层级关联的容器体系，子容器可以访问父容器中的 Bean，但父容器不能访问子容器的 Bean。


### ConfigurableBeanFactory
由大多数bean工厂实现的配置接口。
这个扩展的接口只是为了允许框架内部的即插即用和对bean工厂配置方法的特殊访问。


### AutowireCapableBeanFactory
BeanFactory的扩展接口，实现该接口以便能够自动装配。