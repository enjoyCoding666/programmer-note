### SpringBoot中使用@Retryable注解进行重试

有功能需要重试时，可以使用Spring的 @Retryable 注解.



### 参数含义：

```
value：抛出指定异常才会重试
exclude：指定不处理的异常
maxAttempts：最大重试次数，默认3次
backoff：重试等待策略，默认使用@Backoff，
@Backoff的value(相当于delay)表示隔多少毫秒后重试，默认为1000L；
multiplier（指定延迟倍数）默认为0，表示固定暂停1秒后进行重试。
```



### 依赖包：

```
        <dependency>
            <groupId>org.springframework.retry</groupId>
            <artifactId>spring-retry</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```



### 添加 @EnableRetry 注解

在Application 启动类上，添加 @EnableRetry 注解。

```
@SpringBootApplication(scanBasePackages = {"com.example.demo"})
@EnableRetry
public class DemoApplication {

}
```



### 示例：

```
@Slf4j
@Service
public class RetryServiceImpl {

    @Retryable(value = Exception.class,maxAttempts = 3,backoff = @Backoff(delay = 3000,multiplier = 1.5))
    public void doSomething() {
        log.info("doSomething start.");
        int result = 1/0;
        log.info("doSomething end.");
    }

    /**
     * @Recover 的返回类型，必须跟 @Retryable修饰的方法返回值一致。否则不生效。
     *
     */
    @Recover
    public void recover(Exception e) {
        log.info("retry failed recover");
        log.info("retry Exception:"+e);
        //是否需要入库记录
    }
    
}
```



### 执行结果：

可以看到，@Retryable 修饰的方法执行了3次。仍旧失败后，会执行 @Recover 修饰的方法。

```
2023-10-11 15:21:05.921  INFO 30680 --- [           main] c.e.demo.service.impl.RetryServiceImpl   : doSomething start.
2023-10-11 15:21:08.924  INFO 30680 --- [           main] c.e.demo.service.impl.RetryServiceImpl   : doSomething start.
2023-10-11 15:21:13.425  INFO 30680 --- [           main] c.e.demo.service.impl.RetryServiceImpl   : doSomething start.
2023-10-11 15:21:13.425  INFO 30680 --- [           main] c.e.demo.service.impl.RetryServiceImpl   : retry failed recover
2023-10-11 15:21:13.425  INFO 30680 --- [           main] c.e.demo.service.impl.RetryServiceImpl   : retry Exception:java.lang.ArithmeticException: / by zero
```



### 使用注意：

* @Retryable 底层使用的是 AOP，因此不能在同一个类里调用 @Retryable修饰的方法，不会生效。





### 参考资料：

https://blog.csdn.net/zk86547462/article/details/124597160
