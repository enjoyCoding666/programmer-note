在平常开发中，如果涉及到计算，要求准确的精度，比如单价*数量=总价之类的计算，那么得用到BigDecimal。
###初始化
如下：
```
BigDecimal amount=new BigDecimal("5.33");
```
**注意，最好不要用double类型来初始化，数值并不准确。**
比如
```
BigDecimal amount=new BigDecimal(0.06);
```
那么有可能这个BigDecimal会从0.06变成0.059999999，传入double类型来初始化本身就是不精确的。

如果想用 double 初始化，可以用以下：
```
BigDecimal bigDecimal = BigDecimal.valueOf(0.06);
```


###  BigDecimal转化为String：
toPlainString()。
如果直接用 toString()，结果有可能是科学计数法。最好用 toPlainString()。

```
BigDecimal bigDecimal1=new BigDecimal("230.08");
String str1 = bigDecimal1.toPlainString();
```

### 去掉小数末尾的零
stripTrailingZeros 去掉小数末尾的零。
```
String money = BigDecimal.valueOf(0.06123).setScale(2,RoundingMode.HALF_UP).stripTrailingZeros().toPlainString();
```


### 四舍五入，保留位数
```
setScale(1);//表示保留一位小数，默认用四舍五入方式 
setScale(1,BigDecimal.ROUND_DOWN);//直接删除多余的小数位，如2.35会变成2.3 
setScale(1,BigDecimal.ROUND_UP);//进位处理，2.35变成2.4 
setScale(1,BigDecimal.ROUND_HALF_UP);//四舍五入，2.35变成2.4
setScaler(1,BigDecimal.ROUND_HALF_DOWN);//2.35变成2.3，如果是5则向下舍
```
###加减乘除、绝对值
加法 add()函数     减法subtract()函数
乘法 multipy()函数    除法divide()函数    绝对值abs()函数
重点注意除法，详细参数为
```
pubilc BigDecimal divide(BigDecimal divisor, int scale, int roundingMode)
```
其中divisor为除数，scale为小数点位数, roundingMode表示是否四舍五入。
示例如下：
```
import java.math.BigDecimal;

public class BigDecimalDivideTest {
	public static void main(String[] args) {
		BigDecimal bigDecimal1=new BigDecimal("230.08");
		BigDecimal bigDecimal2=new BigDecimal("20");
                //除法，保留两位，四舍五入
		BigDecimal bigDecimal3=bigDecimal1.divide(bigDecimal2,2,BigDecimal.ROUND_HALF_UP);
		System.out.println(bigDecimal3);
	}
}
```
### 常用的数值
常用的常量有BigDecimal.ZERO表示零。
### 比较大小
**注意，比较BigDecimal大小，不要用equals()比较。**
比较大小，可以用compareTo()函数。
对于BigDecimal变量a和b，进行a.compareTo(b)的返回结果，
如果返回1，表示a>b;
如果返回0，表示a=b;
如果返回-1，表示a<b。




```
import java.math.BigDecimal;

public class BigDecimalTest {
    public static void main(String[] args) {
        BigDecimal detailAmount=new BigDecimal(-11);
        BigDecimal positiveDetailAmount=new BigDecimal(11);
        int result=BigDecimal.ZERO.compareTo(detailAmount);
        int result2=BigDecimal.ZERO.compareTo(positiveDetailAmount);
        System.out.println(result);
        System.out.println(result2);
    }
}

```

参考资料：
https://blog.csdn.net/haiyinshushe/article/details/82721234