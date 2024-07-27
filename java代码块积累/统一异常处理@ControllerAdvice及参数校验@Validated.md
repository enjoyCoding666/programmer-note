### 一、异常处理

有异常就必须处理，通常会在方法后面throws异常，或者是在方法内部进行try catch处理。
**直接throws Exception**
直接throws Exception，抛的异常太过宽泛，最好能抛出准确的异常，比如throws  IOException之类。

```
    User getUserById(Integer id) throws IOException,BusinessException,InterruptedException;
```

如果有多种异常，那么方法后面要throws IOException,InterruptedException又显得冗长。
而且，异常一直向上抛，上层的类还是得处理这些异常。
**try catch捕获异常**
阿里巴巴的java规范中有一条，"最外层的业务使用者，必须处理异常，将其转化为用户可以理解的内容。"
也就是说在Controller层，最好不要又throws Exception继续往上抛了。
但是，如果在Controller层进行大量的捕获异常，可能会出现大量的非常多的try catch代码块。

```
/**
  * 以下这种代码写法很丑。
  */
@PostMapping("/id")
public ResultInfo  getUserById(HttpServletRequest request)  {
	String postData = null;
	try {
		postData = IOUtils.toString(request.getInputStream(), "UTF-8");
	} catch (IOException e) {
		logger.error("IO异常);
	}
	JSONObject postJson = JSONObject.parseObject(postData);
	logger.info("请求中获取的参数为：" + postJson);
	String id=postJson.getString("id");
	User user=new User();
	try{
		user=getUserById(id)
	}catch(BusinessException e){
		logger.error("根据id查找用户发生异常，id:"+{});
	}
	
	//...
	
}
```

这么多的try catch很难看，不建议这样写。

### 二、统一异常处理

**@ControllerAdvice配合@ExceptionHandler，可以很方便地统一处理异常。**
首先是自定义的业务异常类，如下所示：

```
/**
 *  自定义异常。
 * 这里的BusinessException继承于RuntimeException，而非Exception。
 * 如果继承的是Exception，那么在服务层还是得进行异常处理。
 */
public class BusinessException  extends RuntimeException  {
    private static final long serialVersionUID = 1L;
    private String code;
    private String message;


    public BusinessException(String code, String message) {
        super(message);
        this.code = code;
        this.message = message;
    }

    public BusinessException(String message) {
        super(message);
        this.message = message;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    @Override
    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

}
```



### Response 响应类

```
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.annotation.JsonInclude;
import org.apache.commons.lang3.builder.ToStringBuilder;
import org.apache.commons.lang3.builder.ToStringStyle;


/**
 * jackson 注解，忽略掉不相关的属性.如果不是实际项目，只是示例，也可以不加上。
 *
 */
@JsonIgnoreProperties(ignoreUnknown = true)
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Response<T>  {
    private String code = Status.SUCCESS.getCode();
    private String message = Status.SUCCESS.getName();
    private T data;

    public Response() {
    }

    public Response(T data) {
        this.data = data;
    }

    public Response(String code, String message) {
        this.code = code;
        this.message = message;
    }

    public Response(Status status, String message) {
        this.code = status.getCode();
        this.message = message;
    }

    public Response(T data, String code, String message) {
        this.data = data;
        this.code = code;
        this.message = message;
    }

    public static <T> Response<T> success() {
        return new Response<>(null);
    }

    public static <T> Response<T> success(T data) {
        return new Response<>(data);
    }

    public static <T> Response<T> failure() {
        return new Response<>(null, Status.FAILED.getCode(), Status.FAILED.getName());
    }

    public static <T> Response<T> failure(String message) {
        return new Response<>(null, Status.FAILED.getCode(), message);
    }

    public static <T> Response<T> failure(T data) {
        return new Response<>(data, Status.FAILED.getCode(), Status.FAILED.getName());
    }

    public static <T> Response<T> failure(String code, String message) {
        return failure(null, code, message);
    }

    public static <T> Response<T> failure(T data, String code, String message) {
        return new Response<>(data, code, message);
    }

    public static <T> Response<T> failureWithoutData(String errorMsg) {
        return new Response<>(null, Status.FAILED.getCode(), errorMsg);
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public boolean isSuccess() {
        return code.equals(Status.SUCCESS.getCode());
    }



    @Override
    public String toString() {
        return ToStringBuilder.reflectionToString(this, ToStringStyle.JSON_STYLE);
    }

    /**
     * 状态
     */
    public enum Status {
        SUCCESS("200", "成功"),
        FAILED("500", "失败"),

        ;

        private final String code;
        private final String name;

        Status(String code, String name) {
            this.code = code;
            this.name = name;
        }

        public String getCode() {
            return this.code;
        }

        public String getName() {
            return this.name;
        }
    }
}
```





### @ControllerAdvice 和 @ExceptionHandler  统一异常处理

接着是重点，@ControllerAdvice进行统一异常处理。

通过 @ExceptionHandler指定对应的异常处理措施。
如下所示：

```
import org.apache.commons.lang3.StringUtils;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import java.util.ArrayList;
import java.util.List;


@ControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 处理业务异常
     *
     * @param e
     * @return
     */
    @ExceptionHandler(BusinessException.class)
    @ResponseBody
    public Response<Object> handleBusinessException(BusinessException e) {
        log.error("GlobalExceptionHandler handleBusinessException:" + e.getMessage());
        return Response.failure(e.getMessage());
    }


    /**
     * 处理所有接口数据验证异常。对应的是@Validated注解。
     * 注意，MethodArgumentNotValidException 要引入的类是 org.springframework.web.bind.MethodArgumentNotValidException，
     * 不要import错了。
     *
     * @param e
     * @return
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseBody
    public Response<Object> handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        log.error("GlobalExceptionHandler handleMethodArgumentNotValidException e:" + e.getMessage(), e);
        List<String> allMessage = new ArrayList<>();
        for (FieldError fieldError : e.getBindingResult().getFieldErrors()) {
            allMessage.add(fieldError.getDefaultMessage());
        }
        return Response.failure(StringUtils.join(allMessage, ";"));
    }


    /**
     * 处理所有的异常。最好放到最后面，做为兜底。放在前面，就不会再进入具体的异常了。
     *
     * @param e
     * @return
     */
    @ExceptionHandler(Exception.class)
    @ResponseBody
    public Response<Object> handleException(Exception e) {
        log.error("GlobalExceptionHandler handleException e:", e);
        return Response.failure(e.getMessage());
    }

}
```



### 服务层

有了自定义异常，就可以在服务层抛出，直接在方法内部 throw new BusinessException(); 如下示：

```
    @PostMapping("/exception")
    @ResponseBody
    public Response<List<Order>> biz(@RequestBody  Order order) {
        throw new BusinessException("业务异常测试");
    }
```

有了@ControllerAdvice统一异常处理，那么在控制层就无须再处理了。



### 三、参数校验@Validated

@ControllerAdvice除了进行统一异常，还能配合@Validated注解进行参数校验。
Controller层的参数通常都需要检验，经常会看到大量的判空，然后返回错误提示，比如"名字不能为空"之类的提示。
有些人可能会像下面这样写：

```
/**
  *  以下的参数校验实在是太繁杂了。不建议这样写。
  */
@PostMapping("user")
@ResponseBody
public BaseResult getUser()  {

	//名字为空就返回错误提示"名字不能为空"
	if(StringUtils.isEmpty(name)){
		return  new BaseResult( ErrorType.NAME_NOT_NULL );
	}
     //手机号码为空就返回错误提示"手机号码不能为空"
	if(StringUtils.isEmpty(phoneNumber)){
		return  new BaseResult( ErrorType.PHONENUMBER_IS_NULL  );
	}

    // ...
}
```

这些冗长的参数校验，可以通过@Validated注解简化。
如下所示，直接在bean对象上面添加注解：

```
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {

    @NotNull(message = "id不能为空")
    private Integer id;
    @NotBlank(message = "名字不能为空")
    private String userName;
    @Min(value = 18,message = "年龄不能小于18岁")
    private Integer age;
    @NotNull(message = "手机号码不能为空")
    private String phoneNumber;
}
```

其中的类上方注解@Data之类是Lombok注解，详情见：https://www.cnblogs.com/expiator/p/10854141.html
而@NotNull，@Min这些是Validation注解。常见的参数校验注解如下：

```
JSR提供的校验注解：         
@Null   被注释的元素必须为 null    
@NotNull    被注释的元素必须不为 null    
@NotBlank    被注释的元素必须不为 null，不为空格组成   
@AssertTrue     被注释的元素必须为 true    
@AssertFalse    被注释的元素必须为 false    
@Min(value)     被注释的元素必须是一个数字，其值必须大于等于指定的最小值    
@Max(value)     被注释的元素必须是一个数字，其值必须小于等于指定的最大值    
@DecimalMin(value)  被注释的元素必须是一个数字，其值必须大于等于指定的最小值    
@DecimalMax(value)  被注释的元素必须是一个数字，其值必须小于等于指定的最大值    
@Size(max=, min=)   被注释的元素的大小必须在指定的范围内    
@Digits (integer, fraction)     被注释的元素必须是一个数字，其值必须在可接受的范围内    
@Past   被注释的元素必须是一个过去的日期    
@Future     被注释的元素必须是一个将来的日期    
@Pattern(regex=,flag=)  被注释的元素必须符合指定的正则表达式    

```

### 参数校验统一处理

@Validated注解的参数校验同样可以进行统一异常处理。
异常类型为MethodArgumentNotValidException.class 。
在统一异常处理类 GlobalExceptionHandler 中已经有如下代码：

```
    /**
     * 处理所有接口数据验证异常。对应的是@Validated注解。
     * 注意，MethodArgumentNotValidException 要引入的类是 org.springframework.web.bind.MethodArgumentNotValidException，
     * 不要import错了。
     *
     * @param e
     * @return
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseBody
    public Response<Object> handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        log.error("GlobalExceptionHandler handleMethodArgumentNotValidException e:" + e.getMessage(), e);
        List<String> allMessage = new ArrayList<>();
        for (FieldError fieldError : e.getBindingResult().getFieldErrors()) {
            allMessage.add(fieldError.getDefaultMessage());
        }
        return Response.failure(StringUtils.join(allMessage, ";"));
    }
```

### 控制层

只需要在方法参数前面加上注解@Validated ，如下所示：

```
    @PostMapping("/valid")
    @ResponseBody
    public Response<List<Order>> valid(@RequestBody @Validated Order order) {
        Response<List<Order>> response = new Response<>();
        response.setData(orderService.getListByDto(order));
        return response;
    }


```





### 参考资料：

https://blog.csdn.net/kinginblue/article/details/70186586
https://blog.csdn.net/u013815546/article/details/77248003/
