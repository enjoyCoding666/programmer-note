### java常用的工具类/第三方类库
在开发的过程中，有些代码直接用原生的语法写起来比较麻烦。
多掌握一些java常用的工具类、java常用的第三方类库，可以让我们提高效率，代码变得简洁优雅。


### 一、apache commons-lang
apache出品，java开发者经常会用到的工具类库。可以处理字符串、IO、集合等。

#### 依赖：
```
      <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.12</version>
      </dependency>
```

#### StringUtils:

常用的常量：

```
StringUtils.EMPTY: 空字符""
```

处理字符串，常用的方法有
```
boolean isEmpty(CharSequence cs):判断字符串是否为空，字符串为null或者字符串长度为0时返回true

boolean isBlank(CharSequence cs):判断字符串是否为null，空字符串，空格符（空格、换行符、制表符、tab 等）

boolean isNotEmpty(CharSequence cs):判断字符串是否不为空

boolean isNotBlank(CharSequence cs):判断字符串是否不为null，空字符串，空格符
```
示例：
```
String name = "lin";
if (StringUtils.isNotBlank(name)) {

}
```

#### IOUtils：
用于处理IO流。

常见方法：
```
String toString(InputStream input)： 将InputStream 流的内容，转化为字符串。

String toString(InputStream input, String encoding)：将InputStream 流的内容，按照指定的编码，转化为字符串。

List<String> readLines(InputStream input)：将InputStream 流的内容，逐行转化为集合

List<String> readLines(InputStream input, String encoding)：将InputStream 流的内容，按照指定的编码，逐行转化为集合

byte[] toByteArray(InputStream input)： 将InputStream 流的内容，转化为字节数组。
```

#### FileUtils
常用方法：
```
String readFileToString(File file)： 读取文件的内容

List<String> readLines(File file): 逐行读取文件的内容，返回集合。

Collection<File> listFiles(File directory, String[] extensions, boolean recursive) ： 指定文件后缀如txt，遍历文件夹中的文件。
参数extensions是文件后缀，布尔参数recursive表示是否遍历所有的子文件夹。

writeStringToFile(File file, String data) ： 清除文件原来的内容，在文件后面追加内容。

writeStringToFile(File file, String data, boolean append)：布尔参数append为true时，表示不清除文件原来的内容，在文件后面追加内容。 

writeLines(File file, Collection<?> lines)：清除文件原来的内容，在文件后面写入新的内容，逐行地写。

writeLines(File file, Collection<?> lines, boolean append):布尔参数append为true时，表示不清除文件原来的内容，在文件后面逐行追加内容，逐行地写。 
```

示例：
```
    public static void testReadFile() throws IOException {
        //读取文件
        String result = FileUtils.readFileToString(new File("E:\\test.txt"));
        System.out.println(result);
    }

    private static void testReadLines() throws IOException {
        //逐行读取文件
        List<String> lines = FileUtils.readLines(new File("E:\\test.txt"));
        System.out.println(lines);
    }

    public static void testListFile() {
        // 指定文件后缀如txt，遍历文件夹中的文件。注意，此处的test是文件夹名称，没有后缀。
        File directory = new File("E:\\test");
        Collection<File> files = FileUtils.listFiles(directory, new String[]{"txt"}, false);
        System.out.println(files);

    }

    public static void testWriteString() throws IOException {
        File file = new File("E:\\test.txt");
        //清除文件原来的内容，在文件中写入新的内容。
        FileUtils.writeStringToFile(file,"testWriteString");
        String content = FileUtils.readFileToString(file);
        System.out.println(content);
    }

    public static void testWriteStringAppend() throws IOException {
        File file = new File("E:\\test.txt");
        //不清除文件原来的内容，在文件后面追加内容。
        FileUtils.writeStringToFile(file,"testWriteStringAppend", true);
        String content = FileUtils.readFileToString(file);
        System.out.println(content);
    }

    public static void testWriteLine() throws IOException {
        File file = new File("E:\\test.txt");
        // 可以一行行写入文本
        List<String> lines = new ArrayList<>();
        lines.add("test");
        lines.add("testWriteLine");
        FileUtils.writeLines(file,lines);
        String content = FileUtils.readFileToString(file);
        System.out.println(content);
    }

    public static void testWriteLineAppend() throws IOException {
        File file = new File("E:\\test.txt");
        // 可以一行行写入文本
        List<String> lines = new ArrayList<>();
        lines.add("test");
        lines.add("testWriteLineAppend");
        //不清除文件原来的内容，在文件后面追加内容。
        FileUtils.writeLines(file,lines, true);
        String content = FileUtils.readFileToString(file);
        System.out.println(content);
    }

```


### 二、commons-collections：
#### 依赖：
```
<dependency>
	<groupId>commons-collections</groupId>
	<artifactId>commons-collections</artifactId>
	<version>3.4</version>
</dependency>
```

#### CollectionUtils
CollectionUtils类用于处理集合， 常用方法如下：
```
boolean isEmpty(Collection coll): 判断集合是否为空

boolean isNotEmpty(Collection coll)： 判断集合是否不为空

int size(Object object)： 计算集合的个数

```

示例：
```
List<String> list = new ArrayList<>();
int size = CollectionUtils.size(list);

if (CollectionUtils.isEmpty(list)) {
            
}
```

### 三、Objects
jdk自带的类，用于处理对象，判空，转换字符串等。

####  Objects常用方法：
```
String toString(Object o): 将对象转换字符串，由于方法内使用了String.valueOf()，如果对象为 null，返回的是 "null"字符串。

String toString(Object o, String nullDefault)： 将对象转换字符串，如果对象为 null，就返回指定的默认值。

boolean isNull(Object obj)： 判断对象是否为空

boolean nonNull(Object obj)： 判断对象是否不为空
```

示例：
```
String name = Objects.toString( null , "0");
if (Objects.isNull(name)) {

}

```

### Guava常见的代码示例
谷歌出品的，非常实用。包含集合、并发、I/O、散列、缓存、字符串等。
详情见： https://blog.csdn.net/sinat_32502451/article/details/133959892


### HuTool工具类：
详情见： https://blog.csdn.net/sinat_32502451/article/details/133964792

### 参考资料：
https://ifeve.com/google-guava/
https://www.techug.com/post/java-libs-you-should-know.html
https://blog.csdn.net/qq_15717719/article/details/114012722
https://juejin.cn/post/6844904154113146894
