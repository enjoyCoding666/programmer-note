### 使用场景：

可以通过 AOP , 以 控制层 controller 所在的包为切点， 在 controller 的方法前后打印日志，这样就能统计出接口的耗时，还能打印入参，出参，减少重复地打印日志。

如果想统计 dao 层的入参和耗时，也可以用类似的方法。



### 代码示例：

```
@Aspect
@Component
public class RequestLogAop {


    /**
     * 定义切点 切点为 com.example.demo..controller 下所有的类
     *
     * 第一个 controller.* 表示命名为controller的包下的任意类，
     * 第二个 * 表示任意方法。
     * .. 表示匹配任意数量任意类型的参数。
     */
    @Pointcut("execution(* com.example.demo..controller.*.*(..))")
    public void pointcut() {
    }

    /**
     * 环绕通知。在方法的前后打印日志。
     */
    @Around(value = "pointcut()")
    public Object webLogAround(ProceedingJoinPoint joinPoint) throws Throwable {
        // 类名称示例：com.example.demo.controller.OrderTestController
        String className = joinPoint.getTarget().getClass().getName();
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        // 方法名称示例：com.example.demo.controller.OrderTestController#get
        // 可以选中方法后，根据idea快捷键 ctrl+alt+shift+c，复制方法，然后搜索日志。反之亦可。
        String methodName = className + "#" + signature.getMethod().getName();
        long start = System.currentTimeMillis();
        //过滤无法序列化的参数类型
        Object[] objects = filterArgs(joinPoint.getArgs());
        //在方法执行前，打印日志
        log.info("class method: {} , parameter:{}.", methodName, JSON.toJSONString(objects));
        Object proceed = joinPoint.proceed();
        //在方法执行后，打印日志
        //如果还需要打印接口的返回值，就打印 JSON.toJSONString(proceed)
        log.info("class method: {} , takes time: {}ms.",
                methodName, (System.currentTimeMillis() - start));
        return proceed;
    }

    /**
     * 过滤无法序列化的参数类型
     *
     * @param args
     * @return
     */
    private Object[] filterArgs(Object[] args) {
        Object[] results = new Object[args.length];
        for (int i = 0; i < args.length; i++) {
            if (args[i] instanceof ServletRequest || args[i] instanceof ServletResponse || args[i] instanceof MultipartFile) {
                //ServletRequest不能序列化，从入参里排除，否则报异常
                continue;
            }
            results[i] = args[i];
        }
        return results;
    }

}
```





### 显示日志：

```
[INFO ] 2023-11-14 23:41:36 c.example.demo.config.RequestLogAop 43: class method: com.example.demo.controller.OrderTestController#get , parameter:[{"id":1,"orderId":"svsvsdvsvwev","userId":1234}]. 

[INFO ] 2023-11-14 23:41:36 c.example.demo.config.RequestLogAop 46: class method: com.example.demo.controller.OrderTestController#get , takes time: 161ms. 
```


方法名称示例：com.example.demo.controller.OrderTestController#get

如果想根据代码搜索日志，那么可以选中方法后，根据idea的快捷键 ctrl+alt+shift+c，复制方法，然后搜索日志。

如果想根据日志搜索代码，那么复制日志打印出来的类和方法，然后可以用idea的快捷键 双shift ，快速搜索方法.
