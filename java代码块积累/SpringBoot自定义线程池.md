
### Spring/SpringBoot自定义线程池

在 Spring/SpringBoot 中，可以使用 @Configuration 和 @Bean 去设置线程池，用 @Value 去做线程池的参数配置。


### 依赖包：

引用 google 的 guava包。

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>19.0</version>
</dependency>
```


### 线程池配置：

```
import com.google.common.util.concurrent.ThreadFactoryBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.*;



@Configuration
public class ThreadPoolExecutorConfig {

    /**
     * 核心线程数， :冒号后面是默认值
     */
    @Value("${executor.core.pool.size:20}")
    private int corePoolSize;

    /**
     * 最大线程数， :冒号后面是默认值
     */
    @Value("${executor.max.pool.size:100}")
    private int maxPoolSize;

    /**
     * 线程池中空闲线程等待工作的超时时间。也就是没有任务执行时，线程的存活时间。
     */
    @Value("${executor.keep.alive.time:2}")
    private int keepAliveTime;

    /**
     * 阻塞队列长度
     */
    @Value("${executor.blocking.queue.size:200}")
    private int blockingQueueSize;


    /**
     * 使用 @Bean 指定线程池的名称
     *
     */
    @Bean("myExecutor")
    public ThreadPoolExecutor myExecutor() {

        //阻塞队列长度
        BlockingQueue<Runnable> workQueue = new ArrayBlockingQueue<>(blockingQueueSize);

        //创建线程或线程池时指定有意义的线程名称，方便出错时回溯。名称如果过长，会被截断。
        ThreadFactory threadFactory = new ThreadFactoryBuilder()
                .setNameFormat("my-executor-%d").build();
        //可以自定义拒绝策略，也可以不加这个参数，构造方法会直接用默认的拒绝策略。
        //核心业务，可以使用 CallerRunsPolicy 策略，非核心业务，使用 AbortPolicy 策略。
        ThreadPoolExecutor.CallerRunsPolicy abortPolicy = new ThreadPoolExecutor.CallerRunsPolicy();

        return new ThreadPoolExecutor(corePoolSize, maxPoolSize,
                keepAliveTime, TimeUnit.SECONDS, workQueue, threadFactory, abortPolicy);


    }


    /**
     * 不同的业务，有时可以使用不同的线程池、不同的配置。
     *
     */
    @Bean("myExecutor2")
    public ThreadPoolExecutor myExecutor2() {

        //阻塞队列长度
        BlockingQueue<Runnable> workQueue = new ArrayBlockingQueue<>(blockingQueueSize);

        //创建线程或线程池时指定有意义的线程名称，方便出错时回溯。
        ThreadFactory threadFactory = new ThreadFactoryBuilder()
                .setNameFormat("my-executor2-%d").build();
        //可以自定义拒绝策略，也可以不加这个参数，构造方法会直接用默认的拒绝策略
        //核心业务，可以使用 CallerRunsPolicy 策略，非核心业务，使用 AbortPolicy 策略。
        ThreadPoolExecutor.CallerRunsPolicy abortPolicy = new ThreadPoolExecutor.CallerRunsPolicy();

        return new ThreadPoolExecutor(corePoolSize, maxPoolSize,
                keepAliveTime, TimeUnit.SECONDS, workQueue, threadFactory, abortPolicy);
    }

}

```

### 配置参数：

可以写到配置中心里面，如果没有配置中心，放到 propeties 文件也行。

```
executor.core.pool.size=20
executor.max.pool.size=100
executor.keep.alive.time=2
executor.blocking.queue.size=200
```



### 使用线程池：

日志框架自行补充，可以在Service类上面用 lombok的 @Slf4j 注解，也可以用其他的日志。

```
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.concurrent.*;


@Service
@Slf4j
public class MyService {

    @Resource(name = "myExecutor")
    private ThreadPoolExecutor executor;

    public void execute() {
        executor.execute(() -> log.info("线程池执行任务."));

    }

    public void submit() throws Exception {
        Future<String> future = executor.submit(() -> {
            log.info("线程池执行异步任务.");
            return "okk";
        });
        System.out.println("异步任务返回：" + future.get(3, TimeUnit.SECONDS));
    }

}

```

运行，查看日志：

```
INFO 18876 --- [ my-executor-0] com.example.demo.threadpool.MyService    : 线程池执行任务.
```

可以看到 线程池已经执行任务。日志中的 线程名称 是 my-executor 开头的。



把MyService 类中的  (name = "myExecutor") 换成 (name = "myExecutor2") ，再次运行，查看日志：

```
INFO 18876 --- [ my-executor2-0] com.example.demo.threadpool.MyService    : 线程池执行任务.
```

可以看到日志，线程池已经执行任务。 线程名称 变成 了 my-executor2 开头的了。

创建线程或线程池时，指定有意义的线程名称，方便出错时回溯。



### Spring / 普通Java项目的自定义线程池

详情见：https://www.cnblogs.com/expiator/p/17140760.html
