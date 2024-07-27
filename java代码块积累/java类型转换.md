
类型转换虽然很简单，但是还是有些小细节要多注意。

### String转化为int：
```
String test="123";
int number=Integer.parseInt(test);
```
### String转化为Integer:

```
String test="123";
//非数字的字符串，作为valueOf()的参数，会报错：NumberFormatException: For input string
// String test="abc";　　
Integer number=Integer.valueOf(test);
```
注意：不管是使用Integer.parseInt()，还是使用Integer.valueOf()将字符串转换成数字，

如果是非数字的字符串，会报错：NumberFormatException: For input string: ""

另外，Integer类取值和 int 类型取值一致，取值范围是从-2147483648 至 2147483647（-231至 231-1） ，包括-2147483648 和 2147483647。

如果超过了这个范围，也会报错。比如Integer.valueOf("2147483648")，超过了Integer范围。因此会报错： For input string: "2147483648"

更安全的做法是，使用apache包的NumberUtils，如下：

注意：NumberUtils只处理整数，不能用来处理小数。
```
String str="abc";
//str不为数字时，设置默认值为 0
int num = NumberUtils.toInt(str);
//str不为数字时，设置默认值为其他值，比如1
int defaultNum=NumberUtils.toInt(str,1);
```

### Integer转化为int:
可以用 intValue()方法。
```
    Integer num = null;
    int result = num ==null? 0: num.intValue();
```

### int转化为Integer：
```
  int test = 123;
  Integer number=Integer.valueOf(test);
```

### String转BigDecimal：
```
  String str1="2.30";
  BigDecimal bd=new BigDecimal(str1);
```

### String转double ：
```
  double value = NumberUtils.toDouble("4.23");

```

### Double转化为int：
```
Double test=new Double("1.23");　　//Double初始化，最好用String保证精度
int result=test.intValue();
```

### 其他类型转String：
```
//  Object obj= null;
String test=String.valueOf(obj);
```
注意：当String.valueOf()的参数obj为null时，返回值是字符串"null"！！而不是null。

如果希望obj为null时，返回""，可以使用 Objects.toString(object, "")。
或者是 apache-commons-lang包的 ObjectUtils，如下所示：
```
Object object=null;
String str = ObjectUtils.toString(object);　　//object为null时，结果为""
```
如果希望obj为null时，返回null，如下：

ObjectUtils.toString(object,nullStr)，第二参数nullStr表示，当object为null时，方法返回的值。
```
// Object obj=null;
Object object="123";
String str = ObjectUtils.toString(object,null);
//相当于 String str= (object == null) ? null : object.toString();
```

### Integer转double：
使用doubleValue()方法，或者 (double)强制转换。
```
      Integer a= new Integer(5);
      int intvalue=a.intValue();
      double doublevalue=a.doubleValue();
```

### 其他类型转Double：
```
Double rate= Double.valueOf(obj);
```

### 比较小数是否相等。

比较Double是否相等。比较BigDecimal是否相等。

如下所示：
```
double value=1.23;
if (BigDecimal.ZERO.compareTo(BigDecimal.valueOf(value)) == 0) {
      //
}
```
### 比较Double类型的大小：
```
if (Double.valueOf(d1).compareTo(Double.valueOf(d2))<0) {
    //...
}

```
### 比较double类型的大小：

除了用BigDemical的compare()方法，可以直接用Double.doubleToLongBits()的结果值用==，>，<进行比较
```
      if(Double.doubleToLongBits(d1) == Double.doubleToLongBits(d2))){
            //      
      }
```


### double 格式化，保留两位小数：

double 格式化，保留两位小数。

new DecimalFormat().format()有些瑕疵，如果是5，保留两位小数，返回的还是5.

```
        double percent =  8911.333;
        String format=new DecimalFormat("##.##").format(percent);
```

如果要求一定要两位小数，可以用：

```
        double num = 5.459;
//        double num = 5;
        String format = String.format("%.2f", num);
```

### String字符串格式化

可以使用String.format()：

```
String name="userName";
String result="111";
String format = String.format("%s结果为:%s", name, result);
System.out.println(format);
```

### java对象转换
可以使用 BeanUtils.copyProperties

```
BeanUtils.copyProperties(原来的对象, 目标对象);

BeanUtils.copyProperties(原来的对象, 目标对象, 不要复制的属性);
```

