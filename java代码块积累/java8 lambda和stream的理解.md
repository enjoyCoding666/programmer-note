### 一、lambda表达式
语法：
```
(parameters) -> expression
或
(parameters) ->{ statements; }
```
parameters是参数，expression是表达式，statements是代码块。

lambda表达式，其实就是匿名函数。

->左侧是方法参数，参数可以有多个。->右侧是方法内容，也可以直接是方法的返回值。

比如 x->x+5，表示接收参数x，返回x+5。

(x, y) -> x + y ，表示接收参数x和y，返回x+y的和。

lambda表达式，可以作为其他方法的参数。

### :: 双冒号用法
可以通过 :: 来使用类的方法。

语法：
```
类名::方法名
```
比如 Integer::valueOf，就相当于Integer.valueOf()。

System.out::println，就相当于System.out.println();

Person::new，就相当于new Person();

有些lambda表达式，可以用双冒号用法来表示。

比如 x->Integer.valueOf(x) ，可以写成 Integer::valueOf ，

x->x!=null ,可以写成 Objects::nonNull ，

person -> person.getName() ，可写成 Person::getName .
### 创建对象
创建对象Worker,后续会用作示例，如下：
```
public class Worker {

    private String id;

    private Integer age;

    private String name;
    
    public Worker(String id, Integer age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }
    
    //getter()和setter()自行生成
```
### 二、forEach()
forEach() 方法迭代集合中的数据。

* 遍历List：
```
/**
 * 遍历集合
 *
 */
public void forEachDemo() {
	List<String> list=Arrays.asList("abc","def","","123");
	list.forEach(System.out::println);
}
```
* 遍历Map:

```
public void foreachTest() {
    List<Worker> list = new ArrayList<>();
    list.add(new Worker("123",18,"lin"));
    list.add(new Worker("456",28,"chen"));
    list.add(new Worker("789",38,"wu"));
    Map<String, Integer> map = new HashMap<>();
    //forEach()内的方法参数为work，执行的操作是将Worker对象的id作为key，age作为value
    list.forEach(worker -> map.put(worker.getId(),worker.getAge()));
    //遍历map，打印key和value
    // map.forEach((key,value)->System.out.println(key+","+value));
    map.forEach((key,value)-> {
            System.out.println(key+","+value);
            if ("456".equals(key)) {
                System.out.println("TargetValue:"+value);
            }
        });
}
```
* 注意：

在forEach()中不能使用continue、break，即使使用return也不会终止lambda。
如下：
```
public void forEachContinueTest() {
        List<Worker> list = new ArrayList<>();
        list.add(new Worker("123",18,"lin"));
        list.add(new Worker("456",28,"chen"));
        list.add(new Worker("789",38,"wu"));
        Map<String, Integer> map = new HashMap<>();
        list.forEach(worker -> map.put(worker.getId(),worker.getAge()));
        map.forEach((key,value)->{
            System.out.println(key+","+value);
            if ("456".equals(key)) {
                System.out.println("Over");
                //在forEach()中使用continue会报错"Continue outside of loop"
//                continue;   
                //在forEach()中使用return并不能中止lambda表达式
                return;      
            }
        });

    }
```
运行结果为：
```
123,18
456,28
Over
789,38
```

## 三、stream流
stream()流，可以非常方便地处理集合数据。包括filter()、map()、sorted()、forEach()等操作。

stream() − 为集合创建串行流。parallelStream() − 为集合创建并行流。

流的操作又分为：

（1）中间操作：filter(Predicate<T>), map(Function(T, R), limit, sorted(Comparator<T>), distinct，flatMap;

（2）终端操作：只有终端操作才能产生输出，包括：allMatch, anyMatch, noneMatch, findAny, findFirst, forEach, collect, reduce, count

### map()
* map() 方法用于映射每个元素到对应的结果。
```
/**
 * 映射
 */
public void mapDemo(){
	List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
	// 获取对应的平方数。distinct()用于去重。
	List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
	squaresList.forEach(System.out::println);
}
```
* mapToInt()、mapToLong()用于转换流的数据类型：
  mapToInt()将映射转成int类型，转化后还可以接上max()，min()，sum()等函数。
```
public void mapToIntDemo() {
	List<String> numberList= Arrays.asList("3", "2","2","3");
	//将集合中的字符串转化数字，并进行加一操作，最后求和。
	int sum = numberList.stream().mapToInt(Integer::valueOf).map(x->x+1).sum();
	System.out.println(sum);
}
```

* BigDecimal 求和：
```
BigDecimal bb =list.stream().map(Money::getAmount).reduce(BigDecimal.ZERO,BigDecimal::add);
```

### filter()
```
/**
 * 过滤
 */
public void filterDemo(){
	List<String> list=Arrays.asList("abc","def","","123");
	//筛选出不为空的数据
	List<String> filterList = list.stream().filter(StringUtils::isNotEmpty).collect(Collectors.toList());
	filterList.forEach(System.out::println);
}

```
如果要用filter过滤多个条件，可以如下表示：

比如要判断集合中的任意对象person不为空，且personName也不为空：
```
List<Person> filterList = list.stream()
                              .filter(p-> (p!=null && p.getPersonName()!=null ))
                              .collect(Collectors.toList());

```
### limit()
limit 方法用于获取指定数量的流。

### sorted()
sorted 方法用于对流进行排序。
```
public void sortedDemo() {
	List<Integer> list=Arrays.asList(1,3,5,7,2,4,6);
	List<Integer> numberList = list.stream().sorted().collect(Collectors.toList());
	numberList.forEach(System.out::println);

}

```
倒序输出，示例如下：
```
public void sortedReverseDemo() {
	List<Integer> list=Arrays.asList(1,3,5,7,2,4,6);
	List<Integer> numberList = list.stream().sorted(Comparator.reverseOrder()).collect(Collectors.toList());
	numberList.forEach(System.out::println);
}
```
### count()
```
/**
 * 计数
 */
public void countDemo() {
	List<String> list=Arrays.asList("abc","def","","123",null);
	//使用apache-common中的StringUtils，可以避免空指针
	long count = list.stream().filter(StringUtils::isNotEmpty).count();
	System.out.println(count);
}
```
### collect()
collect() 可以将流转化为集合以及字符串等其他形式。

* collect(Collectors.toList()) ： 表示将流转化为List集合。
```
public void listStringtoInteger() {
    List<String> strList=new ArrayList<>();
    strList.add("1");
    strList.add("2");
    strList.add("abc");
    //将list转化为流，然后过滤掉非数字的字符串，再将字符串转化为数字，最后转化回list。
    List<Integer> integerList = strList.stream().filter(StringUtils::isNumeric).map(Integer::valueOf).collect(Collectors.toList());
    integerList.forEach(System.out::println);
}
```
* collect(Collectors.joining(","))：表示将流转化为逗号分隔的字符串。

示例如下：
```
/**
 *  将集合转化成符号分隔的字符串
 *
 */
public void collectJoiningDemo(){
	List<String> list=Arrays.asList("abc","def","","123");
//		String contents=String.join(",",list);
	//表示通过 ，号分隔
	String contents = list.stream().filter(StringUtils::isNotEmpty).collect(Collectors.joining(","));
	System.out.println(contents);
}

```
* 还有将流转化为Map的Collectors.toMap。

//以下的toMap(Worker::getId, Worker::getName)表示将流转化为Map，键为Worker对象的id属性，值为Worker对象的name属性。
Map<String, String> map = list.stream().collect(Collectors.toMap(Worker::getId, Worker::getName));

### Collectors.toMap
在Collectors.toMap()中，如果没有对重复key的处理 ，那么key重复时会报错"Duplicate key"， 而且value为null时会报错"空指针异常"。
```
    public static void toMapTest() {
        List<Worker> list = new ArrayList<>();
        list.add(new Worker("123",18,"lin"));
        list.add(new Worker("123",18,"li"));
        list.add(new Worker("000",18,null));
        //如果没有对重复key的处理 ，那么key重复时会报错"Duplicate key"， 而且value为null时会报错"空指针异常"。
        Map<String, String> map = list.stream().collect(Collectors.toMap(Worker::getId, Worker::getName));
        for (Map.Entry<String, String> entry : map.entrySet()) {
            System.out.println(entry.getKey()+","+entry.getValue());
        }
    }

```
* 解决办法：
  加上对重复 key的处理。
  (k1,k2)->k2。表示如果 map中如果key1和key2重复，value就取key2对应的value。
```
Map<String, String> map = list.stream().collect(Collectors.toMap(Worker::getId, Worker::getName, (k1,k2)->k2));
```

参考资料：https://www.jianshu.com/p/aeb21dea87e0
### allMatch、anyMatch、noneMatch
用于匹配stream流中的数据。

allMatch: 都满足条件才返回true;
anyMatch: 有一个满足就返回true;
noneMatch: 都不满足才返回true;

api如下：
```
boolean anyMatch(Predicate<? super T> predicate);
```
示例如下：
```
boolean b = Arrays.asList(1, 2, 3, 1).stream().anyMatch(p -> p > 2); //返回true
```
### findAny、findFirst
查找，返回的结果是一个Optional对象。

Optional的理解，详情见： https://www.cnblogs.com/expiator/p/12442629.html

api如下：
```
findAny:     Optional<T> findAny()
findFirst:     Optional<T> findFirst()
```
示例如下：
```
Arrays.asList(1, 2, 3, 4, 1).stream().filter(p -> p > 2).findAny() //输出Optional[3]
Arrays.asList(1, 2, 3, 4, 1).stream().filter(p -> p > 2).findFirst() //输出Optional[3]
```
### reduce()
归约操作，可以将流中的数据，按lambda表达式或者双冒号用法反复地操作，最终得到一个值。

比如.reduce( (x,y) -> x + y) 会对流中所有的数据进行相加，得到结果。也可以写成 .reduce(Integer::sum)。

reduce在求和、求最大最小值等方面都可以很方便，注意其返回的结果是一个Optional对象。
```
public class StreamReduce {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);
        int sum = list.stream().map( x -> x*x ).reduce( (x,y) -> x + y).get();
        System.out.println(sum);
      
//          传统的代码表现方式如下：
//          List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);
//          int sum = 0;
//          for(Integer n : list) {
//              int x = n * n;
//              sum = sum + x;
//          }
//          System.out.println(sum);

    }
}

    
```

参考资料：
https://www.cnblogs.com/whuxz/p/9570613.html
