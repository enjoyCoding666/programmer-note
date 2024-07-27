
### 大量判空的代码

实际中，对象不判空会导致空指针异常。

为了规避为指针，不得不写出这种非常冗长又丑陋的空指针判断。

```
public void tooMuchNull(Worker worker) {
	if (worker != null) {
		Address address=worker.getAddress();
		if (address != null) {
			String city=address.getCity();
		}
	}
}
```



### Optional\<T\>

Optional 可以用来处理空指针。

**Optional\<T\>包含的对象value可能非null，也可能为null。**

常建的构建Optional\<T\>对象的方法，有ofNullable(T value)、of(T value)。

构建时，最终都会调用Optional的构造方法Optional(T value)。

而常见的判断Optional结果的方法有，orElse()、ifPresent()、get()、empty()、map()、flatMap()。


### api图

如下图所示：
![](https://i-blog.csdnimg.cn/blog_migrate/1f759e269a70ec6696ebad1549d65cd0.jpeg)



### 代码示例：

* Bean对象：

```
public class Worker {
    private String id;

    private Integer age;

    private String name;

	//此处忽略构造方法、getter()、setter()等方法。

}
```




* orElse():

```
/**
 * orElse(默认值)，如果Optional<T>封装的对象不存在值，则返回默认值。
 */
public void orElseDemo() {
//		Worker worker1=new Worker("123",18,"lin");
	Worker worker1 = null;
	Worker worker2 = new Worker("456", 28, "chen");
	//如果worker1不为null，则orElse返回worker1,否则返回默认值worker2
	Worker result = Optional.ofNullable(worker1).orElse(worker2);

	//相当于以下代码：
//		if (worker1 != null) {
//			result = worker1;
//		} else {
//			result = worker2;
//		}
	System.out.println(result.getName() + "," + result.getAge());
}
```

* orElseGet():

```
/**
 * orElseGet()，如果Optional<T>封装的对象不存在值，则执行Supplier函数式。
 * orElseGet(Supplier<? extends T> other)，返回的类型必须和Optional封装的对象类型一致。
 */
public void ofElseGetDemo() {
	String name1 = null;
	String name2 = "lin";
	//orElseGet(Supplier<? extends T> other)，返回的类型必须和Optional封装的对象类型一致。
        String result = Optional.ofNullable(name1).orElseGet(()-> name2+"def");
	System.out.println(result);
}

```

* of():

```
/**
 * of(对象)，如果封装的对象为空，则会报出空指针异常
 */
public void ofDemo() {
//		Worker worker1=new Worker("123",18,"lin");
	Worker worker1 = null;
	Worker worker2 = new Worker("456", 28, "chen");
	Worker result = Optional.of(worker1).orElse(worker2);
	System.out.println(result.getName() + "," + result.getAge());
}
```

* isPresent():

```
/**
 * isPresent()表示如果Optional<T>封装的对象不为空，就返回true。
 */
public void isPresentDemo() {
	Worker worker1 = new Worker("123", 18, "lin");
	Optional<Worker> workerOpt = Optional.ofNullable(worker1);
	//这种写法比较丑，可以直接用后续会说到的ifPresent()方法代替。
	boolean isPresent = workerOpt.isPresent();
	if (isPresent) {
		System.out.println(workerOpt.get().getName());
	}
	
	
	//以上代码，相当于：
//		if (worker1 != null) {
//			System.out.println(worker1.getName());
//		}
}
```

* ifPresent(lambda):

```
/**
 * ifPresent(lambda)表示如果对象不为null，则会执行对应的lambda语句。
 */
public void ifPresentDemo() {
	Worker worker1 = new Worker("123", 18, "lin");
//		Worker worker1=null;
	List<String> nameList = new ArrayList<>();
	Optional.ofNullable(worker1).ifPresent(worker -> nameList.add(worker.getName()));
	
	
	//上面这句代码的作用相当于以下代码：
//		if (worker1 != null) {
//			nameList.add(worker1.getName());
//		}
	nameList.forEach(System.out::println);
}

```

* map(lambda)：

map的参数里面是lambda表达式，会从Optional对象中进行映射，提取和转换值。
map是一个非常实用的方法。用得也比较多。
比如：

```
String name = "";
if (worker!=null) {
    name = worker.getName();
}
```

**这种大量的判空在项目开发中随处可见。可以使用 Optional的 map方法替换。**

```
String name = Optional.ofNullable(worker).map(Worker::getName).orElse("");
```



map的 其他示例：

```
public  void mapDemo() {
    String str=" test ";
    Optional.ofNullable(str).map(String::trim)
	.filter(t -> t.length()> 1)
	.ifPresent(s->{
	    s+="1234";
	    System.out.println(s);
	});
	
	
//相当于以下代码：
//	if (str != null) {
//	    str=str.trim();
//	    if (str.length() > 1) {
//		str+="1234";
//		System.out.println(str);
//	    }
//	}
}
```

* flatMap(lambda):

flatMap()的参数是lambda表达式，返回值是Optional。

```
/**
 * flatMap()，如果Optional封装对象不为空，就会执行对应的mapping函数，返回Optional类型的值，否则就返回一个空的Optional对象。
 * 通过flatMap()，可以不断地返回Optional对象，一直进行链式调用。非常重要~
 */
public void flatMapDemo() {
	//假设 Worker类还有一个属性 Address对象.
	Address address = new Address("中国", "广东", "深圳");
	Worker worker = new Worker("123", 18, "lin", address);
	String city = Optional.ofNullable(worker)
			.flatMap(Worker::getAdress)
			.flatMap(Address::getCity)
			.orElse("default");
	System.out.println(city);
}
```

* orElseThrow():

```
/**
 * orElseThrow(),如果Optional封装的对象为空，就会抛出对应的异常。
 */
public void orElseThrowDemo() {
//		Worker worker = new Worker("123", 18, "lin");
	Worker worker = null;
	Worker result = Optional.ofNullable(worker)
			.orElseThrow(IllegalArgumentException::new);
	System.out.println(result.getName());
}
```

* filter(lambda):

```
public void filterDemo() {
	Worker worker = new Worker("123", 18, "lin");
	Optional<Worker> result = Optional.ofNullable(worker)
			.filter(worker1 -> worker1.getAge() > 20);
	//如果符合条件(比如，年龄大于20)则为true，不符合则为false
	System.out.println(result.isPresent());
}
```



### 区别：

*  of() 和 ofNullable() 的区别：

这两个方法都可以创建包含值的 Optional。
不同之处在于如果你把 null值作为参数传递进ofNullable()，而传递null作为参数时，of() 方法会抛出 NullPointerException。

* orElse()和orElseGet()的区别：
  orElse(默认值)，表示如果有值则返回该值，否则返回传递给它的默认值。

orElseGet(lambda表达式)会在有值的时候返回值，如果没有值，它会执行作为参数传入的函数式接口(返回类型必须和Optional封装的对象是同一种类型)，并将返回其执行结果。

需要特别注意的是：

Optional的orElse()若方法不是纯计算型的，有与数据库交互或者远程调用的，都应该使用orElseGet() 。

orElse()无论前面Optional容器是null还是non-null，都会执行orElse里的方法，orElseGet()并不会。

详情参见：https://blog.csdn.net/weixin_30437337/article/details/95443798

* isPresent()和ifPresent(lambda)的区别：

看方法名is开头，就可以知道isPresent()返回的是布尔值。而ifPresent()则是如果对应的值存在，就会执行函数式接口。

* flatMap(lambda)和map(lambda)的区别：

值不为空时，两者都会执行参数中的函数式接口。

而flatMap()返回值是Optional，通过不断地产生Optional，可以进行链式调用。







### 代码地址：

https://github.com/enjoyCoding666/JavaProject/tree/master/src/main/java/com/java8/optional



### 参考资料：

https://www.cnblogs.com/zhangboyu/p/7580262.html

https://juejin.im/post/5e66ecdc518825490d126a16
