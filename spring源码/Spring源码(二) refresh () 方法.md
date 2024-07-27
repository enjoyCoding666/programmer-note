### 版本号：

SpringBoot 版本： 2.0.6.RELEASE

Spring 版本： 5.0.9.RELEASE

###  Spring源码(一) 如何阅读 Spring 源码

详情见： https://blog.csdn.net/sinat_32502451/article/details/140155044


### ApplicationContext 和 BeanFactory

在阅读 refresh () 之前，可以简单了解下 ApplicationContext 和 BeanFactory。

详情见： https://blog.csdn.net/sinat_32502451/article/details/140247662

### refresh () 方法

refresh()方法 负责初始化 ApplicationContext 容器。在调用refresh()方法之后，要么全部实例化，要么根本不实例化。

refresh()方法的路径： org.springframework.context.support.AbstractApplicationContext#refresh

```
    public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {
            //准备刷新
            prepareRefresh();

            //通知子类初始化/更新内部的bean factory
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

            // 准备BeanFactory
            prepareBeanFactory(beanFactory);

            try {
                // 后置处理。允许在BeanFactory初始化后进行修改，抽象类方法为空，可以继承后修改BeanFactory
                postProcessBeanFactory(beanFactory);

                // 调用上下文中注册为bean的工厂处理器。。
                invokeBeanFactoryPostProcessors(beanFactory);

                // 实例化并调用所有注册的Bean后置处理器
                registerBeanPostProcessors(beanFactory);

                // 为上下文初始化messageSource
                initMessageSource();

                // 为上下文初始化eventMulticaster
                initApplicationEventMulticaster();

                // 初始化其他特殊的beans
                onRefresh();

                // 检查监听器bean并注册
                registerListeners();

                // 实例化所有非懒加载的单例。这个方法，可以重点看看。
                finishBeanFactoryInitialization(beanFactory);

                // 发布相应的事件
                finishRefresh();
            }

            catch (BeansException ex) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
                }

                // 销毁已创建的singleton
                destroyBeans();

                // 重置标记
                cancelRefresh(ex);

                // 向调用方传播异常
                throw ex;
            }

            finally {
                //重置Spring core中的缓存                
                resetCommonCaches();
            }
        }
    }

```

### BeanFactory

- BeanFactory  生成的具体路径：

org.springframework.context.support.AbstractApplicationContext#refresh   ---->
org.springframework.context.support.AbstractApplicationContext#obtainFreshBeanFactory   ---->

org.springframework.context.support.AbstractApplicationContext#refreshBeanFactory     ---->

org.springframework.context.support.AbstractRefreshableApplicationContext#refreshBeanFactory      ---->

org.springframework.context.support.AbstractRefreshableApplicationContext#createBeanFactory

- refreshBeanFactory () 方法：

```
    @Override
    protected final void refreshBeanFactory() throws BeansException {
        if (hasBeanFactory()) {
            destroyBeans();
            closeBeanFactory();
        }
        try {
            //重点看这一块代码
            DefaultListableBeanFactory beanFactory = createBeanFactory();
            beanFactory.setSerializationId(getId());
            customizeBeanFactory(beanFactory);
            loadBeanDefinitions(beanFactory);
            synchronized (this.beanFactoryMonitor) {
                this.beanFactory = beanFactory;
            }
        }
        catch (IOException ex) {
            throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
        }
    }
```

- createBeanFactory() 方法：

BeanFactory 是在这个方法生成的。

createBeanFactory() ： 为此上下文创建一个内部 BeanFactory。

```
    // 为此上下文创建一个内部 BeanFactory
    protected DefaultListableBeanFactory createBeanFactory() {
        return new DefaultListableBeanFactory(getInternalParentBeanFactory());
    }
```

- 设置 BeanFactory  的属性：

```
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        // 告知内部bean工厂使用上下文的类加载器等
        beanFactory.setBeanClassLoader(getClassLoader());
        beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
        beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

        // 使用上下文回调配置bean工厂
        beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
        beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
        beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
        beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
        beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
        beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
        beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

        beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
        beanFactory.registerResolvableDependency(ResourceLoader.class, this);
        beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
        beanFactory.registerResolvableDependency(ApplicationContext.class, this);

        beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

        if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
            beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
            // Set a temporary ClassLoader for type matching.
            beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
        }

        // 注册默认环境相关的bean
        if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
            beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
        }
        if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
            beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
        }
        if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
            beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
        }
    }
```


### 参考资料：

https://baijiahao.baidu.com/s?id=1724327365978008424

https://cloud.tencent.com/developer/article/2129293
