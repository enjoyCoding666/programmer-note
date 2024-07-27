
### postfix completion的使用

详情见：[https://blog.csdn.net/sinat_32502451/article/details/133026276](https://blog.csdn.net/sinat_32502451/article/details/133026276)


**注意： 这些关键字，不用全部都打出来，只打几个关键的字母也可以！！！**


### 自定义 postfix completion



### if 语句：

* if 语句：

key：

```
if
```

表达式：

```
if ($EXPR$$END$) { 
        
}
```



* if 语句判空：

在 idea 里面已经有了 key 为 nn 的 postfix，用自定义的 key，会好记一些。

(1)key：

```
ifNotNull
```

表达式：

```
if ($EXPR$ != null) {
    $END$;        
}
```

(2)key：

```
ifnull
```

表达式：

```
if ($EXPR$ == null) {
    $END$;        
}
```



### 字符串String

- 字符串判空：

key:

```
ifIsBlank
```

表达式：

```
if (StringUtils.isBlank($EXPR$)) {
    $END$        
}
```

key:

```
ifIsNotBlank
```

表达式：

```
if (StringUtils.isNotBlank($EXPR$)) {
    $END$        
}
```

- 字符串比较

key：

```
ifEquals
```

表达式：

```
if ($END$.equals($EXPR$)) {
     
}
```

在 END 这个地方输入了 变量后，直接 shift+Enter 跳转到下一行。



* 字符串转换为 list：

key：

```
strToList
```

表达式：

```
List<String> list = Arrays.stream($EXPR$$END$.split(",")).map(String::trim).collect(Collectors.toList());
```

END 放到 EXPR  后面 ，有一些容错性，搞错了也能删掉重新输入字符串。


* 使用Optional 对字符串判空：

这个 key是 idea 自带的，不需要新增。

```
opt
```

表达式：

```
Optional.ofNullable()
```





### List 集合：

- 初始化list：

key:

```
list
```

表达式：

```
List<$EXPR$> $END$ = new ArrayList<>();
```



通过这个表达式，只要输入 String.list ，就能生成：

```
List<String>  = new ArrayList<>();
```




- 集合判空：

key :

```
ifListIsEmpty
```

表达式：

```
if (CollectionUtils.isEmpty($EXPR$)) {
    $END$
}
```

key :

```
ifListIsNotEmpty
```

表达式：

```
if (CollectionUtils.isNotEmpty($EXPR$)) {
    $END$
}
```

* Optional 对 list 集合判空：
  key:
```
OptionalList
```
表达式：
```
Optional.ofNullable($EXPR$).orElse(new ArrayList<>())
                .stream()$END$
```



* list 转换为 String：

key:

```
listToStr
```

表达式：

```
String $END$ = String.join(",",$EXPR$);
```







### Map

* map初始化：

key:

```
map
```

表达式：

```
Map<String, $EXPR$$END$> map = new HashMap<>();
```



* map循环：

key:

```
mapfor
```

表达式：

```
for (Map.Entry<String ,  $END$> entry : $EXPR$.entrySet()) {
    
}
```



### MybatisPlus

* LambdaQueryWrapper：

key:

```
LambdaQueryWrapper
```

表达式：

```
LambdaQueryWrapper$END$<$EXPR$> queryWrapper = new LambdaQueryWrapper<>();
queryWrapper.eq($EXPR$::

```



### Json


*  Java 对象转换为 JSON字符串：

key:

```
objToJsonStr
```

表达式：

```
String $END$ = JSON.toJSONString($EXPR$);
```

*  JSON 字符串转换成Java对象：

key:

```
jsonStrToObj
```

表达式：

```
JSON.parseObject( $EXPR$ ,   $END$.class);
```

*  JSON 字符串转换成JSONObject对象：

key:

```
jsonStrToJsonObj
```

表达式：

```
   JSONObject $END$ = JSON.parseObject($EXPR$);
```

*  Java对象转换为 JSONObject ：

key:

```
objToJsonObj
```

表达式：

```
JSONObject $END$ = (JSONObject) JSON.toJSON($EXPR$);
```
