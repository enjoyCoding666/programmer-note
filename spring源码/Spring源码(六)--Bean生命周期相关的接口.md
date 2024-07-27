### Bean生命周期接口
Bean生命周期接口相关的这些类和接口，都可以多看看。
Bean工厂实现应该尽可能支持标准的Bean生命周期接口。
整套初始化方法及其标准顺序为:

```
* 
 * <li>BeanNameAware's {@code setBeanName}
 * <li>BeanClassLoaderAware's {@code setBeanClassLoader}
 * <li>BeanFactoryAware's {@code setBeanFactory}
 * <li>EnvironmentAware's {@code setEnvironment}
 * <li>EmbeddedValueResolverAware's {@code setEmbeddedValueResolver}
 * <li>ResourceLoaderAware's {@code setResourceLoader}
 * (only applicable when running in an application context)
 * <li>ApplicationEventPublisherAware's {@code setApplicationEventPublisher}
 * (only applicable when running in an application context)
 * <li>MessageSourceAware's {@code setMessageSource}
 * (only applicable when running in an application context)
 * <li>ApplicationContextAware's {@code setApplicationContext}
 * (only applicable when running in an application context)
 * <li>ServletContextAware's {@code setServletContext}
 * (only applicable when running in a web application context)
 * <li>{@code postProcessBeforeInitialization} methods of BeanPostProcessors
 * <li>InitializingBean's {@code afterPropertiesSet}
 * <li>a custom init-method definition
 * <li>{@code postProcessAfterInitialization} methods of BeanPostProcessors
```


### BeanNameAware

```
/**
 *  该接口由希望在beanFactory中知道其bean名称的bean实现。
 */
public interface BeanNameAware extends Aware {

    /**
     *  在创建此bean的bean工厂中设置bean的名称。
     */
    void setBeanName(String name);

}
```


### BeanClassLoaderAware

```
/**
 *  允许bean知道bean类加载器的回调；也就是，当前bean工厂用来加载bean类的类加载器。
 */
public interface BeanClassLoaderAware extends Aware {

    /**
     * 将bean类加载器提供给bean实例的回调。
     */
    void setBeanClassLoader(ClassLoader classLoader);

}
```