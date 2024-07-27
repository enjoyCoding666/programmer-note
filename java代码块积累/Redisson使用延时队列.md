
### 延时队列

在开发中，有时需要使用延时队列。

比如，订单15分钟内未支付自动取消。



### jdk延时队列

如果使用 jdk自带的延时队列，那么服务器挂了或者重启时，延时队列里的数据就会失效，可用性比较差。



### Redisson延时队列

可以使用Redisson的延时队列。

### Redisson的配置

详情见：
[https://blog.csdn.net/sinat_32502451/article/details/133799192](https://blog.csdn.net/sinat_32502451/article/details/133799192)

注意：不要使用太低的 版本，低版本有内存泄露的问题。可以使用 3.17 及以上的版本。



### Redisson延时队列的初始化

可以把 delayedQueue 的初始化，放到 Spring的 @Bean 中管理。这样不用频繁地初始化队列，不用浪费太多系统资源。

后面就可以通过 @Resource 在其他类中注入延时队列的类。

```
@Configuration
public class RedissonBeanConfig {


    @Resource
    private RedissonClient redissonClient;

    /**
     * 延时队列，参数为以下的阻塞队列，注意 @Qualifier 后面的对象名不要写错
     */
    @Bean(name = "redisDelayedQueue")
    public RDelayedQueue<String> redisDelayedQueue(@Qualifier("redisBlockingQueue") RBlockingQueue<String> blockingQueue) {
        return redissonClient.getDelayedQueue(blockingQueue);
    }

    /**
     * 阻塞队列
     */
    @Bean(name = "redisBlockingQueue")
    public RBlockingQueue<String> redisBlockingQueue() {
        return redissonClient.getBlockingQueue(队列名称);
    }


}



```






### 在延时队列中添加任务

```
	/**
     * 延迟时间
     */
    @Value("${delay.time:5}")
    private Integer delayTime;

    @Resource
    private RDelayedQueue<String> redisDelayedQueue;

    @Resource
    private RBlockingQueue<String> redisBlockingQueue;

    public void addDelayQueue(String orderId) {
       //blockingDeque和 delayedQueue ， 最好通过上面讲解的 @Configuration 初始化后,  @Resource 注入使用
        //RBlockingDeque<String> blockingDeque = redissonClient.getBlockingDeque("orderQueue");
        //RDelayedQueue<String> delayedQueue = redissonClient.getDelayedQueue(blockingDeque);
        
        //在延时队列中添加任务，5秒后生效
        redisDelayedQueue.offer(orderId, delayTime , TimeUnit.SECONDS);
        log.info("addDelayQueue orderId:" + orderId);
    }
```





### 取出延时队列中的任务

* 第一次方式(稍差些)：

实现 ApplicationRunner，  while (true) 轮询。。

while (true) 轮询，万一发生错误没有继续执行，有可能会影响业务。

使用 take() 方法取出延时队列中的任务，如果延时队列中没有任务，会一直阻塞，直到队列中添加了任务。


```
@Component
@Order(1)
@Slf4j
public class DelayQueueRunner implements ApplicationRunner {

    @Resource
    private RedissonClient redissonClient;

    @Resource
    private RDelayedQueue<String> redisDelayedQueue;

    @Resource
    private RBlockingQueue<String> redisBlockingQueue;

    @Override
    public void run(ApplicationArguments args) {
        //blockingDeque和 delayedQueue ， 最好通过上面讲解的 @Configuration 初始化后,  @Resource 注入使用
        //RBlockingDeque<String> blockingDeque = redissonClient.getBlockingDeque("orderQueue");
        while (true) {
            try {
            	//如果队列中没有值，会一直阻塞
                String orderId = redisBlockingQueue.take();
                //异步处理。业务逻辑，此处忽略。
                

            } catch (Exception e) {
                log.error("take error.", e);
            }
        }
    }
}

```



* 第二种方式(较好)：

**注意：如果项目中有用到定时任务的中间件，可以使用中间件，设置每隔1-2秒去取出延时队列中的任务。定时任务如果间隔太久，有可能影响消费的效率 。消费得太慢，数据会堆积。**
**while (true) 轮询，万一发生错误没有继续执行，有可能会影响业务。**
如果使用了 定时任务去调用任务，就不需要实现ApplicationRunner 了，也不需要 while (true) 轮询。

使用poll()方法，指定超时时间，从阻塞队列中取数。如果在 指定时间没有取到值，就会超时，如果超时，取到的值为空。

```
/**
  *  使用定时任务，调用该方法。
  */
@Component
public class DelayQueueServiceImpl  {

    @Resource
    private RedissonClient redissonClient;

    @Resource
    private RDelayedQueue<String> redisDelayedQueue;

    @Resource
    private RBlockingQueue<String> redisBlockingQueue;

    /**
     * 超时时间
     */
    @Value("${poll.time:2}")
    private Integer pollTimeOut;

    private static final String ORDER_QUEUE = "orderQueue";

    public void pullQueue() {
        log.debug("pullQueue blockingDeque.");
        //blockingDeque和 delayedQueue ， 最好通过上面讲解的 @Configuration 初始化后,  @Resource 注入使用
        //RBlockingDeque<String> blockingDeque = redissonClient.getBlockingDeque(ORDER_QUEUE);

        try {
        	//使用poll()方法，指定超时时间，从阻塞队列中取数。如果在 指定时间没有取到值，就会超时，如果超时，取到的值为空。
            String orderNo = redisBlockingQueue.poll(pollTimeOut, TimeUnit.SECONDS);
            if (StringUtils.isBlank(orderNo)) {
                return;
            }
            log.info("pullQueue blockingDeque orderNo:{}.", orderNo);
            //异步处理。业务逻辑，此处忽略。

        } catch (InterruptedException e) {
            log.error("pullQueue InterruptedException error.",  e);
            Thread.currentThread().interrupt();
        } catch (Exception e) {
            log.error("pullQueue error.",  e);
        }
    }
    
    
}


```





### 日志：

不断在延时队列中拉取数据，由于队列中没有数据，所以该方法会先阻塞。

接着调用 addDelayQueue()方法，往队列中添加数据，观察日志，可以发现 5秒后，取到队列中的数据。



```
[2023-10-12 21:30:54.725]  INFO  c.c.m.c.controller.DelayQueueController  [line: 54] addDelayQueue orderId:12345
[2023-10-12 21:30:59.821]  INFO  c.c.m.c.controller.DelayQueueRunner [line: 72] DelayQueue get orderId:12345
```



### 参考资料：

https://huaweicloud.csdn.net/637ee8b6df016f70ae4c959b.html
https://blog.csdn.net/sinat_32502451/article/details/133799192
https://blog.csdn.net/qq_27818157/article/details/107514319
https://blog.csdn.net/u012228523/article/details/132339547
