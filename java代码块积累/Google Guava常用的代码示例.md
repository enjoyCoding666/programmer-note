### Google Guava
谷歌出品的，非常实用。包含集合、并发、I/O、散列、缓存、字符串等。

#### 依赖：
```
<dependency>
	<groupId>com.google.guava</groupId>
	<artifactId>guava</artifactId>
	<version>18.0</version>
</dependency>
```
#### Joiner
Joiner可以连接字符串。

常用方法：
```
Joiner on(String separator)： 指定连接的符号

Joiner skipNulls() : 跳过null

String join(@Nullable Object first, @Nullable Object second, Object... rest)： 连接多个字符串
```
示例如下：
```
    public static void testJoiner() {
        //使用;来连接多个字符串，跳过null
        Joiner joiner = Joiner.on(";").skipNulls();
        String join = joiner.join("Harry", null, "Ron", "Hermione");
        System.out.println(join);

        List<Integer> list = Arrays.asList(1, 5, 7);
        //list使用逗号连接
        String result = Joiner.on(",").join(list);
        System.out.println(result);
    }
```
#### Splitter
Splitter可以用来分隔字符串。

常用方法：
```
Splitter on(final String separator)：指定Splitter的分隔符。

Iterable<String> split(final CharSequence sequence) : 拆分字符串。

List<String> splitToList(CharSequence sequence)：拆分字符串，并转换成集合。
注意，splitToList使用了@Beta注解，带有@Beta注解的类或方法可能会发生变化。
它们可以随时以任何方式进行修改，甚至可以删除。不推荐使用。
Splitter 返回List，可以使用 Lists.newArrayList(splitter.split(string))

Splitter omitEmptyStrings()：  从结果中自动忽略空字符串。

Splitter fixedLength(final int length): 按固定长度拆分；最后一段可能比给定长度短，但不会为空。
```
示例如下：
```
    public static void testSplit() {
        Splitter splitter = Splitter.on(",");

        System.out.println("按分隔符,去拆分字符串.如下：");
        Iterable<String> splits = splitter.split("a,,b,c");
        //遍历splitter的结果为 Iterable
        for (String split : splits) {
            System.out.println(split);
        }
    }

    public static void testSplitList() {
        System.out.println("split返回 list，如下：");
        // 输出结果 [a, , b, c]
        List<String> splitList = Lists.newArrayList( Splitter.on(",").split("a,,b,c"));
        System.out.println(splitList);
    }

    public static void testSplitOmitEmpty() {
        System.out.println("忽略空字符串，如下：");
        // 输出结果 [a, b, c]
        Iterable<String> omit = Splitter.on(",").omitEmptyStrings().split("a,,b,c");
        System.out.println(omit);
    }
    
    public static void testFixedLength() {
        System.out.println("按固定长度拆分；最后一段可能比给定长度短，但不会为空。如下：");
        //输出结果为 a,,b 以及 ,c
        Iterable<String> fixedLengths = Splitter.fixedLength(4).split("a,,b,c");
        for (String split : fixedLengths) {
            System.out.println(split);
        }
    }

```

#### Lists类
常用方法：
```
ArrayList<E> newArrayList(): 初始化一个ArrayList

ArrayList<E> newArrayList(E... elements) ：初始化一个ArrayList，并添加多个元素

List<List<T>> partition(List<T> list, int size) ： list分页，参数 list 表示被分割的集合， size表示每一页的数量。可以分批处理。非常实用。
```

示例：
```
    public static void testPartition() {
        //初始化list
        List<String> list = Lists.newArrayList("lin","chen","wu","zhang","qiu");
        //分页，每页2个元素
        List<List<String>> partitionList = Lists.partition(list, 2);
        if (CollectionUtils.size(partitionList) > 0) {
            //下标是从0开始的,先判断集合的size，避免数组越界。
            List<String> partList = partitionList.get(0);
            //结果为 [lin, chen]
            System.out.println(partList);
        }
    }
```

#### Maps类
常用方法:
```
HashMap<K, V> newHashMap() ： 初始化Map
```
示例如下：
```
Map<String, String> map = Maps.newHashMap();
```

#### 参数检查：
```
public class CheckArgumentDemo {


    public static void main(String[] args) {
        checkNotNull();
    }

    /**
     * 判断空指针
     */
    public static void checkNotNull() {
        String str = null;
        Preconditions.checkNotNull(str);
    }

    /**
     *  检查参数，不满足条件，就给出错误提示
     *
     */
    public static void checkArgument() {
        int age = 17;
        Preconditions.checkArgument(age>=18, "年龄未满18~");
    }


    /**
     *  检查参数，不满足条件，就按格式说明符替换后，给出错误提示
     *
     */
    public static void checkArgumentWithFormat() {
        String name = "lin";
        int age = 17;
        Preconditions.checkArgument(age>=18, "%s年龄为%s，年龄未满18",name, age);
    }



}

```

#### Table
Table是Guava中的一种数据结构，两个key对应一个value，相当于表格，某行某列对应一个值。
* 常用方法：

```
//以下泛型<R, C, V>,R表示Row(行)，C表示Column(列), V表示Value(值)。
<R, C, V> HashBasedTable<R, C, V> create()： 创建一个Table

V put(R rowKey, C columnKey, V value)： 在Table中指定行，指定列，放入对应的值。

V get(@Nullable Object rowKey, @Nullable Object columnKey)： 获取Table指定行，指定列的值。行和列不能为空。

Map<C, V> row(R rowKey)： 获取某一行的所有数据。

Map<R, V> column(C columnKey);  获取某一列的所有数据。

Set<Cell<R, C, V>> cellSet(); 获取Table的所有数据
```
示例：
```
    public static void testTable() {
        Table<String, String, Integer> table = HashBasedTable.create();

        table.put("第一行", "第一列", 11);
        table.put("第一行", "第二列", 12);
        table.put("第二行", "第一列", 13);
        //获取Table的所有数据
        Set<Table.Cell<String, String, Integer>> cellSet = table.cellSet();
        System.out.println("cellSet：" + cellSet);
        for (Table.Cell<String, String, Integer> cell : cellSet) {
            String rowKey = cell.getRowKey();
            String columnKey = cell.getColumnKey();
            Integer value = cell.getValue();
            System.out.println("row:" + rowKey + " ,column:" + columnKey + ",value:" + value);
        }
        System.out.println("cellSet结束：" + cellSet);
        
        //获取对应行，对应列的数据
        Integer value = table.get("第一行", "第二列");
        System.out.println(value);

        //获取第一行的数据
        Map<String, Integer> rowMap = table.row("第一行");

        Map<String, Integer> columnMap = table.column("第一列");
        System.out.println(columnMap);

    }
```

