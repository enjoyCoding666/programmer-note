### FastJson依赖包：
```
    <!-- fastjson依赖 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.76</version>
    </dependency>
```

### 示例Bean
先创建Person类，如下：
```
public class Person {

    private int age;

    private String fullName;

    @JSONField(format="yyyy-MM-dd hh:mm:ss")
    private Date dateOfBirth;

    // 忽略 getters & setters
}
```


## Java 对象转换
###  Java 对象转换为 JSON字符串
JSON.toJSONString() 将 Java 对象(或集合)转换换为 JSON字符串。
假设person为Java对象，则如下：
```
    String jsonStr= JSON.toJSONString( person);
```
List等集合转换为JSON字符串，也可以用 JSON.toJSONString.


### Java 对象 转 JSONObject：

```
JSONObject json = (JSONObject) JSONObject.toJSON(person);
```

## JSONObject 转换
### JSONObject 转换成 JSON字符串

```
JSONObject json = new JSONObject();
//忽略json的取值过程
String jsonStr = json.toString();
```

### JSONObject 转换成 Java对象
```
JSONObject json = new JSONObject();
json.put("age", 1);
Person person = JSONObject.parseObject(json.toJSONString(), Person.class);
```

##  JSON 字符串转换
###  JSON 字符串转换成Java对象。
parseObject 方法可以将 JSON 字符串转换成Java对象。
假设JSON字符串为jsonStr，如下：
```
    Person newPerson = JSON.parseObject( jsonStr, Person.class);
```
### JSON 字符串转换成JSONObject对象
```
   JSONObject paramJson= JSON.parseObject(jsonStr);
```

### list转JSON字符串：

map转JSON字符串也可以用。

```
String jsonStr = JSON.toJSONString(list);
```

### JSON字符串转list：

```
List<String> strList = JSON.parseObject(jsonStr, new TypeReference<List<String>>() {});
if (strList  == null ) {
	strList = new ArrayList<>();
}
```

### JSON字符串转map：

```
Map<String, Integer> map = JSON.parseObject(jsonStr, new TypeReference<Map<String, Integer>>() {});
```

### JSON字符串转泛型对象：

其他的泛型对象，类似于JSON字符串转map，都可以采用以下的TypeReference泛型进行转换。

```
List<Person> resultList = JSON.parseObject(jsonStr, new TypeReference<List<Person>>() {});
```

### java对象的字段指定json字段名称
@JSONField用在java对象的变量上面，这里的name表示的是转换后的JSON字段。
```
    @JSONField(name = "FULL NAME")
    private String fullName;
```

### java对象的Date变量转换成Json并格式化
```
    @JSONField(format="yyyy-MM-dd hh:mm:ss")
    private Date dateOfBirth;
```
@JSONField用在java对象的变量上面，这里的name表示的是转换后的JSON字段，格式则用format处理。

如果项目中使用的是jackSon，就用
```
@JsonProperty(value = "DATE OF BIRTH")
@JsonFormat(pattern = "yyyy-MM-dd hh:mm:ss")
private Date dateOfBirth;
```

### JSONObject获取元素的值
* JSONObject获取字符串：
```
String fullName = json.getString("fullName");
```

* JSONObject获取整型：
```
int age = json.getInt("age");
```


### JSONObject移除元素
可以用remove()方法移除JSONObject的某个键值。
```
	public static void main(String[] args) {
		String str="{\"buyerTaxNo\": \"440301999999980\",\"errorIds\": \"123\"}";
		JSONObject jsonObject= JSON.parseObject(str);
		jsonObject.remove("errorIds");
		System.out.println(jsonObject);
	}    
```

##JSONArray
### JSONArray转化为List
JSONArray格式如下：
```
{
  "clientList": [ "数组元素1","数组元素2" ]
} 
```
解析代码如下：
```
   	  String clientIdJson=orderJson.getString("clientList");
	  List<String> clientIdList= JSONArray.parseArray(clientIdJson,String.class);
```

### List转化为JSONArray：
```
JSONArray jsonArray =JSON.parseArray(JSON.toJSONString(list));
```

### 遍历JSONArray
```
   for(int i=0;i<jsonArray.size();i++){
        JSONObject jsonobject=jsonArray.getJSONObject(i);
            
    }
```

### 参考资料：
http://www.runoob.com/w3cnote/fastjson-intro.html
