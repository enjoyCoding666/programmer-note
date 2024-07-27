
### LocalDate转Date
```
    /**
     *
     * LocalDate转Date
     * @param localDate
     * @return
     */
    public static Date toDate(LocalDate localDate) {
        return Date.from(localDate.atStartOfDay().atZone(ZoneId.systemDefault()).toInstant());
    }
```

### LocalDateTime转 Date
```
   /**
     * LocalDateTime转 Date
     * @param localDateTime
     * @return
     */
    public static Date toDate(LocalDateTime localDateTime) {
         return Date.from(localDateTime.atZone( ZoneId.systemDefault()).toInstant());
    }
```

### LocalDateTime转String
```
    /**
     * LocalDateTime转String
     * @param localDateTime
     * @param pattern 格式，类似 yyyy-MM-dd HH:mm:ss
     * @return
     */
    public static String formatToString(LocalDateTime localDateTime, String pattern) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
        return localDateTime.format(formatter);
    }


```

### String转LocalDateTime
```
    /**
     * String转 LocalDateTime
     *
     * @param dateTimeStr 日期的字符串
     * @param pattern 格式，类似 yyyy-MM-dd HH:mm:ss
     * @return
     */
    public  static LocalDateTime toLocalDateTime(String dateTimeStr, String pattern) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
        return LocalDateTime.parse(dateTimeStr, formatter);
    }

```

### String转localDateTime
```
    /**
     * String转localDateTime
     *
     * @param dateTimeStr 日期的字符串
     * @param pattern 格式，类似 yyyy-MM-dd HH:mm:ss
     * @return
     */
    public  static LocalDateTime toLocalDateTime(String dateTimeStr, String pattern) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
        return LocalDateTime.parse(dateTimeStr, formatter);
    }

```

### String转LocalDateTime
```
    /**
     * String转LocalDateTime
     *
     * @param dateTimeStr 日期的字符串
     * @param pattern 格式，类似 yyyy-MM-dd HH:mm:ss
     * @return
     */
    public  static LocalDateTime toLocalDateTime(String dateTimeStr, String pattern) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
        return LocalDateTime.parse(dateTimeStr, formatter);
    }

```

### Date转LocalDateTime
```
    /**
     * Date转LocalDateTime
     * @param date 日期
     * @return
     */
    public static LocalDateTime toLocalDateTime(Date date) {
        return LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
    }
```


### Date转String
```
    /**
     * Date转String
     * @param date  日期
     * @param pattern 格式，类似 yyyy-MM-dd HH:mm:ss
     * @return
     */
    public static String formatToString(Date date, String pattern) {
        if (date == null) {
            return "";
        }
        LocalDateTime localDateTime = LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
        return localDateTime.format(formatter);
    }
```


### String转Date
```
 /**
     * String转Date
     * @param dateTimeStr 字符串
     * @param pattern 格式，类似 yyyy-MM-dd HH:mm:ss
     * @return
     */
    public static Date toDate(String dateTimeStr, String pattern) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
        LocalDateTime localDateTime = LocalDateTime.parse(dateTimeStr, formatter);
        return Date.from(localDateTime.atZone( ZoneId.systemDefault()).toInstant());
    }

```

### Hutool 日期转换
包括：Date转String、String转Date等。

https://blog.csdn.net/sinat_32502451/article/details/133964792


### 参考资料：
LocalDateTime的理解：

https://blog.csdn.net/sinat_32502451/article/details/138199632
