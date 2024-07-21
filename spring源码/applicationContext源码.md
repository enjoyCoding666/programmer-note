
### ApplicationContext

### ApplicationContext 和 BeanFactory

可以先简单了解下 ApplicationContext 和 BeanFactory。

详情见： https://blog.csdn.net/sinat_32502451/article/details/140247662

### ApplicationContext 接口继承图：

![请添加图片描述](https://i-blog.csdnimg.cn/direct/fe8cbc52b5ac4ad7b400dc2ac4f6d724.png)可以看到 ApplicationContext 间接继承了 BeanFactory 。


### ApplicationContext  的功能：

- 访问应用程序组件的BeanFactory方法。继承自ListableBeanFactory。
- 加载文件资源的能力。继承自org.springframework.core.io.ResourceLoader接口。
- 向已注册的监听器发布事件的能力。继承自ApplicationEventPublisher接口。
- 解析消息的能力，支持国际化。继承自MessageSource接口。

### ApplicationContext  源码：

```
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
        MessageSource, ApplicationEventPublisher, ResourcePatternResolver {

    /**
     * 返回此应用上下文的唯一id
     */
    @Nullable
    String getId();

    /**
     * 返回应用上下文
     * 
     */
    String getApplicationName();

    /**
     * 为此上下文返回一个友好名称
     * 
     */
    String getDisplayName();

    /**
     * 返回首次加载此上下文时的时间戳
     */
    long getStartupDate();

    /**
     * 返回上级的上下文
     * 
     */
    @Nullable
    ApplicationContext getParent();

    /**
     * 为此上下文公开 AutowireCapableBeanFactory 功能。
     */
    AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;

}

```





