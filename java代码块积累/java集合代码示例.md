
# 一、List
## ArrayList
### 使用List时，最好初始化容量。
ArrayList的默认容量为10，每次扩容增0.5倍，假如要放置100个元素，需要多次扩容。
```
   List<String> list=new ArrayList<>(100);
```
### String数组转List<String>
使用Arrays.asList。
```
  String[] stryArray=new String[]{"str1","str2","str3"};  
  List<String> list=Arrays.asList(strArray);
```

### String(以逗号隔开)转List<String>：
```
public static List<String> stringToList(String strs){
    if (strs==null) {
        return null;
    }
    String[] str = strs.split(",");
    return Arrays.asList(str);
}
```
String(以逗号隔开)转List<String>，更简单的做法是使用Stream流：
```
  List<String> idList = Arrays.stream(ids.split(",")).map(String::trim).collect(Collectors.toList());
```
**注意，通过Arrays.asList转换的List，不可以使用add()方法。**如果使用的话，会报错：java.lang.UnsupportedOperationException
可以使用以下方式在新的List中添加数据：
```
    String[] stryArray=new String[]{"str1","str2","str3"};  
    List<String> list=Arrays.asList(strArray);
    
    List<String> strList=new ArrayList<>(list);
    strList.add("test");
```
### list分页 subList()
使用 subList()分页时，要注意返回的集合是没有实现 Serializable 接口的， 不可以进行序列化。
正确的做法是：
```
new ArrayList<>( list.subList());
```
如下：
```
List<String> list = new ArrayList<>();
list.add("5");
list.add("4");
List<String> resultList = new ArrayList<>( list.subList(0, 1));
```

### List<String>转成String数组
使用toArray()方法。
```
    List<String> list = new ArrayList<String>(2);
    list.add("test1");
    list.add("test2");
    String[] array = new String[list.size()];
    array = list.toArray(array);
```
### List转String
用逗号隔开的，可以使用String.join(",",list)。也可以使用其他的分隔符。
如下：
```
    List<String> list=new ArrayList<>();
    list.add("shen");
    list.add("zhen");
    list.add("shi");
    String words= String.join(",",list);
    System.out.println(words);
```
### List<Integer>转int[]数组
使用java8的stream()。
```
        int[] arr = list.stream().mapToInt(Integer::valueOf).toArray();
```
### int[]数组转List
```
int[] arr = {4, 5, 3, 6, 2, 5, 1};
List<Integer> list = Arrays.stream(arr).boxed().collect(Collectors.toList());
```

### Integer数组转list：

```
        Integer[] arr = {1, 2, 3, 4, 5};
        List<Integer> list = Arrays.asList(arr);
```


### list 中的最大值/最小值：

```
        Integer min = list.stream().min(Integer::compare).orElse(0);
        Integer max = list.stream().max(Integer::compare).orElse(0);
```


### 数组降序：
```
        Integer[] arr = new Integer[]{1, 2, 4, 5, 8, 10};
        Arrays.sort(arr , Collections.reverseOrder());
```
### list判断集合个数是否为空。
```
    if (list!=null && list.size==0 ){
        //...
    }
```
更推荐的做法是：
使用apache的工具类CollectionUtils.isEmpty()。
```
    if (CollectionUtils.isNotEmpty(list)) {

    }
```

### Optional对集合进行判空：
```
    public void optionalList() {
        List<String> list = new ArrayList<>();
        list.add("abc");
        list.add(null);
        list.add("xyz");
        Optional.ofNullable(list).orElse(new ArrayList<>())
                .stream().filter(StringUtils::isNotBlank).forEach(System.out::println);

        //作用类似以下代码
//        if (CollectionUtils.isNotEmpty(list)) {
//            for (String str : list) {
//                if (StringUtils.isNotBlank(str)) {
//                    System.out.println(str);
//                }
//            }
//        }

    }
```

### Optional取集合中的第一条数据

```
    public void optionalFirst() {
        List<String> list = new ArrayList<>();
        list.add("abc");
        list.add(null);
        list.add("xyz");

        //集合中的数据，取其中一条。比如数据库返回的数据取一条。
        //findFirst取第一条数据，ifPresent 表示不为空就执行
        Optional.ofNullable(list).orElse(new ArrayList<>())
                .stream().filter(StringUtils::isNotBlank)
                .findFirst().ifPresent(System.out::println);


        //作用类似以下代码
//        if (CollectionUtils.isNotEmpty(list)) {
//            for (String str : list) {
//                if (StringUtils.isNotBlank(str)) {
//                    System.out.println("========"+str);
//                    break;
//                }
//            }
//        }
    }
```



### 删除list中的元素：
想要遍历集合，删除元素，如果使用foreach()，可能会报错"ConcurrentModificationException"。

可以使用 removeIf()，如下：
```
public void removeElement() {
    List<String> list = new ArrayList<>();
    list.add("1");
    list.add("2");
    //removeIf()，内部使用的就是Iterator()。
    list.removeIf(Objects::isNull);
}
```
###  交换集合中元素的位置
可以使用Collections.swap()方法实例：

Collections.swap(List<?>, int, int) 方法被用于交换在指定列表中的指定位置的元素。

### list集合根据对象字段去重
Book::getAuthor，其中的 Book是类名，getAuthor()是字段对应的get方法。
```
        ArrayList<Book> list = bookList.stream().collect(
                Collectors.collectingAndThen(Collectors.toCollection(
                        () -> new TreeSet<>(Comparator.comparing(Book::getAuthor))), ArrayList::new));
```

### list根据对象属性分组
使用Collectors.groupingBy分组
```
    public  void listGroupBy(){
        List<Person> list = new ArrayList<>();
        Map<String, List<Person>> listMap = list.stream()
                .collect(Collectors.groupingBy(this::getGroup));
        for (Map.Entry<String, List<Person>> entry : listMap.entrySet()) {
            String key = entry.getKey();
            List<Person> value = entry.getValue();
            //其他逻辑
        }

    }

    public  String getGroup(Person person) {
        if (person.getAge() >= 18 ) {
            return "1";
        } else {
            return "2";
        }
    }
```


### List转Map：
比如User对象有两个属性name和age, 以下的User::getName表示 getName()方法，User::getAge表示getAge()方法。
User对象的List，转化成 key为属性name,value为属性age的 Map。
```
    //如果value是其他类型，就把Integer换成其他类型。
    Map<String, Integer> map = new HashMap<>();
    if (CollectionUtils.isNotEmpty( list)) {
        map = list.stream().collect(Collectors.toMap(User::getName, User::getAge, (key1, key2) -> key2));
    }
```
其中的User::getName表示User对象的getName()方法得到的值。
Collectors.toMap(User::getName, User::getAge, (key1, key2) -> key2)表示这个 map的key为 user.getName()，而value为 user.getAge() .
如果需要把对象作为value，那么就是：
```
Collectors.toMap(User::getName, user-> user , (key1, key2) -> key2)
```
而 (key1, key2) -> key2 则表示如果 map中如果key1和key2重复，value就取key2对应的value。
注意，如果不加上 (key1, key2) -> key2 ，那么当key1和key2重复时会报错"Duplicate key"， 而且value为null时会报错"空指针异常"。

#### LinkedList
* 在增删比较多的场景下，使用LinkedList。

####  返回空的List，不要返回null。
返回null，容易导致空指针异常。
可以使用Collections.emptyList()，表示的是空集合。
示例如下：
```
    public List<BillFiles> queryBillFiles(BillFiles billFiles) {
                //以下是一个简单的数据库查询
        List<BillFiles> billFilesList=billFilesMapper.queryBillFiles(billFiles);
        if(billFilesList==null) {
            billFilesList= Collections.emptyList();
        }
        return billFilesList;
    }
```

### 创建只有一个元素的 List:
可以使用 Collections.singletonList.
```
        List<String> list = Collections.singletonList("test");
        List<Integer> integerList = Collections.singletonList(1);
```

# 二、Set
## HashSet
### 集合去重：可以使用Set不重复的特性进行去重。
将集合作为HashSet()构造方法的参数。
```
HashSet(Collection<? extends E> c)
```
Set的size和原来集合的size相同，说明没有重复数据。也可以用contains()判断。
示例如下：
```
        List<String> list=new ArrayList<>();
        list.add("123");
        list.add("456");
        Set<String> set = new HashSet<>(list);
        for (String s : set) {
            System.out.println(s);
        }
```
也可以放入map的values集合：
```
Map<Integer, Integer> map = new HashMap<>();
Set<Integer> set = new HashSet<>(map.values());

if( map.size() == set.size()){

}
//if(set.contains()){}
```


# 三、Map
## HashMap
### 遍历HashMap
如下：
```
        Map<Integer,String> map=new HashMap<Integer,String>();
        map.put(1,"banana");
        map.put(2,"apple");
        //遍历entrySet()返回的Set,其中每一个元素都是Map.Entry类型的，再通过getKey()、getValue()获取键值对
        //以下的Map为接口名，map为Map接口声明的变量。
        for (Map.Entry<Integer, String> entry : map.entrySet()) {
            System.out.println("键："+entry.getKey() + "，值： " + entry.getValue());
        }

```
### map.keySet()：获取所有的key
### map.values() ：获取所有的value


### map.getOrDefault：如果map的key不存在，返回默认值。
使用map.getOrDefault(Object key, V defaultValue)。
比如，用key存储字符串，用value统计次数。如下：
```
Map<String, Integer> map = new HashMap<>();
String word="s";
int count=map.getOrDefault(word, 0);
map.put(word,  count+ 1);
```


### putIfAbsent
使用 putIfAbsent() , 如果 key存在，不会将值添加到 map 中 。如果key不存在，会将值添加到 map 中。

```
        Map<String, Integer> map = new HashMap<>();
        map.put("apple", 1);
        map.put("banana", 2);

        //key存在，不会将值添加到 map 中
        map.putIfAbsent("apple", 3);
        //key不存在，会将值添加到 map 中
        map.putIfAbsent("orange", 4);

        System.out.println(map);
 ```



### 使用compute()，对map中的k和v进行计算。
```
compute(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction)
```
第一个参数是key，第二个参数是匿名函数式，(k,v)-> 后面就是对key和value的具体处理逻辑。
示例如下：
```
Map<String, Integer> map = new HashMap<>();
String word="s";
Integer count=map.compute(word,(k,v)->{
    if (v==null){
        return 0;
    }
    return v+1;
});

```

### computeIfAbsent
computeIfAbsent 用于判断一个key是否存在于Map中。
key存在，返回key对应的值。
key不存在，会把key和lambda表达式的value添加到map中，并返回。

格式：
```
map.computeIfAbsent(key, k -> defaultValue);
```
示例：
```
        Map<String, Integer> map = new HashMap<>();
        map.put("apple", 1);
        map.put("banana", 2);

        //key不存在，会把key和lambda表达式的结果值添加到map中，结果为3
        Integer value1 = map.computeIfAbsent("orange", k -> 3);
        //key存在，返回key对应的值，也就是返回banana的值2.
        Integer value2 = map.computeIfAbsent("banana", k -> 4);

        System.out.println(value1);
        System.out.println(value2);

        System.out.println(map);
```

