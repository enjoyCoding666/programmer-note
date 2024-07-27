
### 限流算法
https://blog.csdn.net/sinat_32502451/article/details/139223748

### 注意：
RateLimiter限流属于单体版的限流，如果是高并发的分布式系统，需要用分布式限流。


### Maven依赖包：
```
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>18.0</version>
        </dependency>
```
### RateLimiter限流

* RateLimiter初始化：
```
   RateLimiter limiter = RateLimiter.create(5);
```

* RateLimiter令牌桶的方法：
```
create():每秒创建多少令牌;
acquire()：获取一个令牌;
acquire(int permits)：获取指定个数的令牌;　　　　　
tryacquire()：尝试获取令牌;
tryacquire():尝试获取一个令牌，如果获取不到立即返回;
tryacquire(int permits, long timeout, TimeUnit unit):尝试获取permits个令牌，如果获取不到等待timeout时间;
```

### 示例：
```
    public static void runRateLimiter() {
        Long start = System.currentTimeMillis();
        // 每秒产生10个令牌，也就是说每秒最多执行10个任务
        RateLimiter limiter = RateLimiter.create(10);
        for(int i = 1; i < 100; i ++ ) {
            // 请求RateLimiter, 超过permits(也就是create方法的参数，比如上面的10)会被阻塞，然后等待获取令牌
            limiter.acquire();
            System.out.println("acquire:" + i );
        }
        Long end = System.currentTimeMillis();
        System.out.println("限流后总耗时:" + (end - start));
    }
```



### 参考资料：
https://www.iteye.com/blog/jinnianshilongnian-2305117
https://www.jianshu.com/p/5d4fe4b2a726
