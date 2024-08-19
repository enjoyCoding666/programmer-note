### Feign常用的注解：

- name/value：指定FeignClient的名称，如果项目使用了Ribbon，name属性会作为微服务的名称，用于服务发现
- contextId：指定beanID
- url: url一般用于调试，可以手动指定@FeignClient调用的地址
- decode404:当发生http 404错误时，如果该字段位true，会调用decoder进行解码，否则抛出FeignException
- configuration: Feign配置类，可以自定义Feign的Encoder、Decoder、LogLevel、Contract
- fallback: 定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，fallback指定的类必须实现@FeignClient标记的接口
- fallbackFactory: 工厂类，用于生成fallback类示例，通过这个属性我们可以实现每个接口通用的容错逻辑，减少重复的代码
- path: 定义当前FeignClient的统一前缀/上下文

### Feign调用 http的 post 请求：

如果不需要 head参数，就不用加 @RequestHeader。

在做服务调用时，一般不会用到 url属性。my.url 是配置的值，后面的: 是默认值。

示例如下：
```
@FeignClient(name = "myFeignService", 
        url = "${my.url:http://ip:端口/}")
public interface MyFeignService {

    @PostMapping(value = "具体的url路径")
    @ResponseBody
    MyResponse post(@RequestBody MyBody myBody,
              @RequestHeader(name = "headParam1", required = true) String headParam1);

}
```
或者是：
```
    @RequestMapping(value = "/v1/xx/xxx", method = RequestMethod.POST,
            headers = {"userName={userName}",  "userId={userId}" }
    )
    Response<User> getUser(@RequestParam("userName") String userName,
                                     @RequestParam("userId") String userId);
```

### Feign调用 http的 Get 请求：
在做服务调用时，一般不会用到 url属性。my.url 是配置的值，后面的: 是默认值。

示例如下：
```
    @FeignClient(name = "myFeignService", 
        url = "${my.url:http://ip:端口/}")
public interface MyFeignService {
    @GetMapping(value = "具体的url路径")
    @ResponseBody
    MyResponse get(@RequestParam (value = "param1") String param1,
		@RequestParam (value = "param2") String param2);
}
```



### 参考资料：
https://cloud.tencent.com/developer/article/2000153
