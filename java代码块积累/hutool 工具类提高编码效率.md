### hutool  工具类

### hutool  依赖

引入 hutool 依赖包。

```
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.6</version>
</dependency>
```



### StrUtil 常用方法：

* StrUtil.equals：

```
String num = "2";
if (StrUtil.equals("2", num)) {
	System.out.println(num);
}
```

### NumberUtil 数字工具类：

* NumberUtil.isNumber 是否为数字
* NumberUtil.isInteger 是否为整数
* NumberUtil.isDouble 是否为浮点数

* NumberUtil.add 针对数字类型做加法

```
public static void add() {
    BigDecimal bigDecimal1 = BigDecimal.valueOf(1.23);
    BigDecimal bigDecimal2 = BigDecimal.valueOf(1.22);

    BigDecimal bigDecimal = NumberUtil.add(bigDecimal1, bigDecimal2);
    System.out.println(bigDecimal);
}
```

* NumberUtil.sub 针对数字类型做减法
* NumberUtil.mul 针对数字类型做乘法
* NumberUtil.div 针对数字类型做除法

* 比较 BigDecimal ：

```
BigDecimal bigDecimal1 = BigDecimal.valueOf(1.23);
BigDecimal bigDecimal2 = BigDecimal.valueOf(1.22);

boolean isGreater = NumberUtil.isGreater(bigDecimal1, bigDecimal2);
System.out.println(isGreater);
```



### 类型转换 常用方法：
* 转换为整型数组:
```
  String[] b = { "1", "2", "3", "4" };
  Integer[] intArray = Convert.toIntArray(b);
  System.out.println(JSON.toJSONString(intArray));
```

* 数组转化为list:
```
  String[] strArr = {"a", "b", "c", "d"};
  List<String> strList = Convert.toList(String.class, strArr);
  System.out.println(strList);  
```



### CollUtil 集合工具类：

* CollUtil.findOne： 查找符合条件的一条数据

```
List<String> list = Arrays.asList("2","3");
String str = CollUtil.findOne(list, num -> StrUtil.equals("2", num));
```

*  CollUtil.get：根据下标，查找list中的数据：
```
  List<String> list = Arrays.asList("1","2","3","4");
  String str = CollUtil.get(list, 0);
  System.out.println(str);
```

* CollUtil.addAll：往集合中添加另一个集合的数据
  自带null的判断，不需要再重复判断
```
  List<String> list = Arrays.asList("1","2","3","4");
  List<String> list2 = null;  
  CollUtil.addAll(list, list2);
  list.forEach(System.out::println);
```

* CollUtil.page：分页。
  pageNo：页码，从0开始计数，0表示第一页
  pageSize：每页的条目数
```
  List<String> list = Arrays.asList("1","2","3","4");
  List<String> pageList = CollUtil.page(0, 2, list);
  pageList.forEach(System.out::println);
```

### DateUtil 日期工具类：
* DateTime
  DateUtil 日期工具类，返回的大部分日期都是 DateTime的。
  hutool的 DateTime继承了 Date，所以可以用 Date 声明。
  此类重写了父类的 toString()方法，返回值为"yyyy-MM-dd HH:mm:ss"格式。


* 字符串转日期：
```
  String dateStr = "2023-03-01";
  Date date = DateUtil.parse(dateStr);
  System.out.println(date);
```
* 字符串转日期。按指定格式：
```
  String dateStr = "2023-03-01 01:21:32";
  Date date = DateUtil.parse(dateStr, "yyyy-MM-dd hh:mm:ss");
  System.out.println(date);
```
* 日期转字符串。按指定格式：
```
  Date date = new Date();
  String dateStr = DateUtil.format(date, "yyyy-MM-dd hh:mm:ss");
  System.out.println(dateStr);
```

* 当月第一天、最后一天

```
//当月第一天
Date beginOfMonth = DateUtil.beginOfMonth(date);
System.out.println("beginOfMonth: " +beginOfMonth);
//当月最后一天
Date endOfMonth = DateUtil.endOfMonth(date);
System.out.println("endOfMonth: " +endOfMonth);
```

* 上个月。
```
//上个月
Date lastMonth = DateUtil.lastMonth();
System.out.println("lastMonth: " + lastMonth);
```
如果是指定日期的上个月，可以使用日期偏移相关的方法。

* 日期偏移，减掉多少天。
  时间偏移，减掉多少个小时。

```
//日期偏移，减去多少天
Date offsetDay = DateUtil.offsetDay(date, -1);
System.out.println("offsetDay: " + offsetDay);
//日期偏移，减去多少个月
Date offsetMonth = DateUtil.offsetMonth(date, -1);
System.out.println("offsetMonth: " + offsetMonth);
//日期偏移，减去多少小时
Date offsetHour = DateUtil.offsetHour(date, -1);
System.out.println("offsetHour: " + offsetHour);
```

时间偏移，还有一些可以直接调用的函数：

```
//昨天
DateUtil.yesterday()
//明天
DateUtil.tomorrow()
//上周
DateUtil.lastWeek()
//下周
DateUtil.nextWeek()
//上个月
DateUtil.lastMonth()
//下个月
DateUtil.nextMonth()

```

* 获取日期的年、月、日

```
int year = DateUtil.year(date);
System.out.println("year: " + year);

//month的枚举，是从0开始的。所以得加上1
int month = DateUtil.month(date) + 1;
System.out.println("month: " + month);

int day = DateUtil.dayOfMonth(date);
System.out.println("day: " + day);
```


* 计算日期相差多少天。

betweenDay(Date beginDate, Date endDate, boolean isReset) 有三个参数，

解释如下：

```
	 * 有时候我们计算相差天数的时候需要忽略时分秒。
	 * 比如：2016-02-01 23:59:59和2016-02-02 00:00:00相差一秒
	 * 如果isReset为{@code false}相差天数为0。
	 * 如果isReset为{@code true}相差天数将被计算为1
	 * </pre>
	 *
	 * @param beginDate 起始日期
	 * @param endDate   结束日期
	 * @param isReset   是否重置时间为起始时间
	 public static long betweenDay(Date beginDate, Date endDate, boolean isReset) 
```

示例如下：

```
Date date1 = DateUtil.parse("2023-04-13 15:06:27", "yyyy-MM-dd HH:mm:ss");
System.out.println("date1: " + date1);

Date date2 = DateUtil.parse("2023-05-13 16:01:03", "yyyy-MM-dd HH:mm:ss");
System.out.println("date2: " + date2);

long betweenDay = DateUtil.betweenDay(date1, date2, true);
System.out.println("betweenDay: " + betweenDay);
```

其他的 betweenMonth() 方法类似，就是 计算日期相差多少个月。。

* 时分秒置零：
```
//修改日期为某个时间字段起始时间。比如以下表示时分秒置零
Date truncate = DateUtil.truncate(date, DateField.DAY_OF_MONTH);
System.out.println("truncate: " + truncate);
```




### IdUtils 生成唯一id
* 生成随机UUID：
```
  String uuid = IdUtil.randomUUID();
  //格式：cbb021c7-cb48-44cd-b8ba-814620ee4340
  System.out.println(uuid);
```
* 生成UUID，没有横线：
```
  String uuid = IdUtil.simpleUUID();
  //格式：a7e0edfb17ac4120a03842f938f88d34
  System.out.println(uuid);
```
* 雪花算法。获取唯一id。
```
  long id = IdUtil.getSnowflakeNextId();
  //格式：1648328806748430336
  System.out.println(id);

  String idStr = IdUtil.getSnowflakeNextIdStr();
  //格式："1648328806752624640"
  System.out.println(idStr);

```

旧版本，使用如下：
```
 String idStr =  IdUtil.getSnowflake(1,1).nextIdStr();
  ```

* 雪花算法。配合终端ID和数据中心ID。生成唯一id。
```
  //workerId     终端ID
  // datacenterId 数据中心ID
  Snowflake snowflake = IdUtil.getSnowflake(1, 1);
  long id = snowflake.nextId();
  //格式：1648328965712121856
  System.out.println(id);

  String idStr = snowflake.nextIdStr();
  //格式：1648328965716316160
  System.out.println(idStr);
```

### BeanUtil 工具类：

常用方法：

```
Map<String, Object> beanToMap(Object bean, String... properties)： bean转map。可选拷贝哪些属性值，默认是不忽略值为null的值的。
toBean(Object source, Class<T> clazz)：Map转Bean。
copyProperties(Object source, Class<T> tClass, String... ignoreProperties)： 按照Bean对象属性创建对应的Class对象，并忽略某些属性。
```



示例如下：

```
Person person = new Person();
person.setAge(18);
person.setName("test");

// bean转map
Map<String, Object> map = BeanUtil.beanToMap(person);
// map转bean
Person person1 = BeanUtil.toBean(map, Person.class);
// 属性拷贝
Person person2 = BeanUtil.copyProperties(person1, Person.class);

System.out.println("map:" + map.toString());
System.out.println("person1:" + person1.toString());
System.out.println("person2:" + person2.toString());
```



### Hutool的 Http 工具类：

* GET 请求：

```
String  response = HttpUtil.get(url);

String  response = HttpUtil.get(url,timeout);

String  response = HttpUtil.get(url,paramMap);

String  response = HttpUtil.get(url,paramMap,timeout);

//body()：结尾的body()是 HttpResponse里面的方法，表示获取响应中的body内容。
String  response = HttpUtil.createGet(url).execute().body();


```

GET请求的示例如下：

```
String url = "xx.xx.xx.xx:8086/order/detail?orderId=123456789";
//发送get请求并接收响应数据
String  response = HttpUtil.createGet(url).execute().body();

//以上代码相当于：
//HttpResponse httpResponse = HttpUtil.createGet(url).execute();
//String response = httpResponse.body();


```



* POST 请求：

body传参形式为json时，需要将json转成字符串，不支持JSONObejct。

可以使用 JSON.toJSONString(json) 将json转化为字符串。

```
String  response = HttpUtil.post(url, bodyStr );

String  response = HttpUtil.post(url, bodyStr , timeout);

//第二个参数paramMap指表单的数据
String  response = HttpUtil.post(url, paramMap, timeout);

String  response = HttpUtil.post(url, paramMap);

//body()：结尾的body()是 HttpResponse里面的方法，表示获取响应中的body内容。
String  response = HttpUtil.createPost(url).body(bodyStr).execute().body();

String  response = HttpUtil.createPost(url).body(bodyStr).execute().body();

String response = HttpRequest.post(url).body(bodyStr).addHeaders(headerMap).form(formMap).timeout(2000).execute().body();

//也可以使用 HttpRequest.post()
String response = HttpRequest.post(url).body(bodyStr).execute().body();
```

常用方法：

```
timeout(int milliseconds)： timeout单位是毫秒，如果想设置2秒超时，传参 2000
addHeaders(Map<String, String> headers)：新增请求头，不覆盖原有请求头
form(Map<String, Object> formMap)：设置map类型的表单数据
body(jsonStr)： body传参形式为json需要将json转成字符串，不支持JSONObejct对象。可以使用 JSON.toJSONString(json) 将json转化为字符串。
contentType()：设置contentType，比如 "application/json;charset=UTF-8"
body()：结尾的body()是 HttpResponse里面的方法，表示获取响应中的body内容。
```

POST 请求的示例如下：

```
//body传参形式为json时，需要将json转成字符串，不支持JSONObejct。可以使用 JSON.toJSONString(json) 将json转化为字符串。
JSONObject json = new JSONObject();
json.put(xx, xx);
String bodyStr = JSON.toJSONString(json);

//发送post请求并接收响应数据
String response = HttpRequest.post(url).body(bodyStr).execute().body();
```



### 参考资料：
https://blog.csdn.net/abst122/article/details/124091375
