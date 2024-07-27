### Spring/SpringBoot 拦截器
### 拦截器的作用：

拦截器，可以进行请求过滤、权限管理、打印日志、数据校验等。

拦截器，可以在请求前、请求后进行处理。



### 代码示例：

* 拦截器 MyInterceptor:

Spring的拦截器，需要实现 HandlerInterceptor 接口。

```
public class MyInterceptor implements HandlerInterceptor {

    /**
     * 前置拦截. 如果返回 false，就不会调用 url 请求
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String requestUri = request.getRequestURI();
        System.out.println("MyInterceptor preHandle: " + requestUri);
        //如果请求以指定字符串开头，就进行拦截。注意不要漏掉第一个/
        if (requestUri.startsWith("/test/interceptor")) {
            //如果返回 false，就不会调用 url 请求
            return false;
        }

        return true;
    }

    /**
     * 后置拦截
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
        System.out.println("MyInterceptor postHandle: " + request.getRequestURI());
    }

    /**
     * 返回给前端渲染后执行
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        System.out.println("MyInterceptor afterCompletion: " + request.getRequestURI());
    }
}


```

* 配置拦截器：

配置拦截器，需要实现 WebMvcConfigurer 接口，通过 addInterceptor 添加拦截器。

可以多次调用 addInterceptor，支持添加多个拦截器。

还可以通过  addPathPatterns 匹配请求的url，excludePathPatterns 排除特定的 url。

```
@Configuration
@Slf4j
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        log.info("WebMvcConfig addInterceptors registry.");
        //通过  addPathPatterns 匹配请求的url，excludePathPatterns 排除特定的 url
        registry.addInterceptor(new MyInterceptor1()).addPathPatterns("/**").excludePathPatterns("/login", "/register");
        //addInterceptor，支持添加多个拦截器。
        registry.addInterceptor(new MyInterceptor2());


    }
}
```



### 参考资料：

https://blog.csdn.net/heihaozi/article/details/131428958
