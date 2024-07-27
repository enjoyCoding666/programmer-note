### LocalDateTime
java8使用了LocalDateTime和DateTimeFormatter。比之前的Date和Carlendar有所改进。
DateTimeFormatter是线程安全的。DateTimeFormatter中很多属性使用了final修饰。

LocalDate: 只能设置仅含年月日的格式，表示没有时区的日期, LocalDate是不可变并且线程安全的
LocalTime: 只能设置仅含时分秒的格式，表示没有时区的时间, LocalTime是不可变并且线程安全的
LocalDateTime: 可以设置含年月日时分秒的格式 ， 表示没有时区的日期时间, LocalDateTime是不可变并且线程安全的

ZoneId: 时区ID，用来确定Instant和LocalDateTime互相转换的规则
Instant: 用来表示时间线上的一个点（瞬时）
Clock: 用于访问当前时刻、日期、时间，用到时区
Duration: 用秒和纳秒表示时间的数量（长短），用于计算两个日期的“时间”间隔


### 构建LocalDateTime对象：
```
	public  void generateLocalDate() {
		LocalDate date = LocalDate.of(2000, 1, 8);
		LocalDateTime dateTime = LocalDateTime.of(2020, 1, 8, 15, 16, 17);
		System.out.println("date：" + date);
		System.out.println("dateTime:" + dateTime);
	}
```

### 获取LocalDateTime的年月日：
```
	public  void getLocalDateTimeMonth() {
		LocalDateTime dateTime = LocalDateTime.of(2020, 1, 8, 15, 16, 17);
		int year=dateTime.getYear();
		int month=dateTime.getMonth().getValue();
		int day=dateTime.getDayOfMonth();
		System.out.println("dateTime对应的年份为："+year);
		System.out.println("dateTime对应的月份为："+month);
		System.out.println("dateTime对应的天数为："+day);
	}
```
### 字符串String转LocalDateTime：
```
	public  void stringToTime() {
		DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
		String datetimeText = "2020-04-01 23:59:59";
		LocalDateTime localDateTime = LocalDateTime.parse(datetimeText, formatter);
		// LocalDateTime的toString()方法，在日期和时间中间加入了"T"字符串
		System.out.println(localDateTime);
	}
```


### LocalDateTime 转字符串String：
```
	public  void localDateTimeToString() {
		DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
		LocalDateTime dateTime = LocalDateTime.of(2020, 1, 8, 15, 16, 17);
		String dateTimeStr = dateTime.format(formatter);
		System.out.println("dateTimeStr为：" + dateTime);
	}
```

### LocalDateTime 转时间戳毫秒
```
      public static long getMillsByLocalDateTime(LocalDateTime localDateTime) {
        long mills = localDateTime.toInstant(ZoneOffset.of("+8")).toEpochMilli();
        return mills;
    }
```

### 时间戳毫秒转LocalDateTime
```
    public static LocalDateTime getLocalDateTimeByMills() {
        long mills = 1640419285000L;
        LocalDateTime result = new Date(mills).toInstant().atOffset(ZoneOffset.of("+8")).toLocalDateTime();
        return result;
    }
```

### Date转Instant：
```
        Instant instant= date.toInstant();
```
### Instant转Date：
```
        Date date = Date.from(instant);
```

### Date转LocalDateTime ：
```
public static void dateToLocalDateTime(Date date) {
        Instant dateInstant = date.toInstant();
        LocalDateTime localDateTime = LocalDateTime.ofInstant(dateInstant, ZoneId.systemDefault());
    }
```

### LocalDateTime转Date：
```
    public static Date parse(LocalDateTime localDateTime) {
         return Date.from(localDateTime.atZone( ZoneId.systemDefault()).toInstant());
    }
```

### LocalDateTime 获取LocalDate：
```
LocalDate localDate = localDateTime.toLocalDate();
```
### LocalDateTime 获取LocalTime：
```
LocalTime localTime = localDateTime.toLocalTime();
```

## LocalDate

### Date转LocalDate：
```
    public void dateToLocalDate(Date date) {
        Instant dateInstant = date.toInstant();
        LocalDate localDate = LocalDateTime.ofInstant(dateInstant, ZoneId.systemDefault()).toLocalDate();
    }
```
### LocalDate转Date：
```
    public static Date parse(LocalDate localDate) {
        return Date.from(localDate.atStartOfDay().atZone(ZoneId.systemDefault()).toInstant());
    }
```

### localDate转String ：
```
public static String localDateToString(Date date) {
        Instant dateInstant = date.toInstant();
        LocalDate localDate = LocalDateTime.ofInstant(dateInstant, ZoneId.systemDefault()).toLocalDate();
        // localDate转String 
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        String localDateStr = localDate.format(formatter);
        System.out.println(localDateStr);
        return localDateStr;
    }
```

### String转LocalDate：
```
public static void stringToLocalDate() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        String datetimeText = "2020-04-01";
        LocalDate localDate = LocalDate.parse(datetimeText, formatter);
        System.out.println(localDate);
    }
```
### LocalDate获取年月日:
```
    int year = localDate.getYear();
    int monthValue = localDate.getMonthValue();
    int dayOfMonth = localDate.getDayOfMonth();

```
### LocalDate修改年月日：
```
    public static void changeYearMonthDay() {
        LocalDate localDate = LocalDate.now();
        //修改年份
        LocalDate withYearResult = localDate.withYear(2020);
        //修改月份
        LocalDate withMonthResult = localDate.withMonth(5);
        //修改日
        LocalDate withDayResult = localDate.withDayOfMonth(20);
        System.out.println( withYearResult + "，" + withMonthResult+ "，"+ withDayResult);
    }
```
### LocalDate比较日期大小compareTo：
compareTo()方法
在两边相等（=）的情况下返回 0
在左边大于（>）右边时返回 1
在左边小于（<）右边时返回 -1
```
      public static void localDateCompareTo() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        String datetimeText = "2020-04-01";
        LocalDate localDate = LocalDate.parse(datetimeText, formatter);

        LocalDate nowDate = LocalDate.now();
        System.out.println(localDate);
        System.out.println("nowDate:"+nowDate);

        if (nowDate.compareTo(localDate) < 0) {
            System.out.println("nowDate小于"+ datetimeText);
        } else {
            System.out.println("nowDate大于等于"+ datetimeText);
        }
    }
```
### LocalDate比较日期的前后：
after 只在大于（>）情况下才为true　(相等时不会）
before 只在小于（<）情况下才为true　(相等时不会）

```
public static void isBeforeAfter() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        String datetimeText = "2020-04-01";
        LocalDate localDate = LocalDate.parse(datetimeText, formatter);

        LocalDate nowDate = LocalDate.now();
        System.out.println(localDate);
        System.out.println("nowDate:"+nowDate);

        if (nowDate.isBefore(localDate)) {
            System.out.println(nowDate+ "在"+ datetimeText +"之前");
        } else {
            System.out.println(nowDate+ "在"+ datetimeText +"之后");
        }
    }
```

### 获取日期差距 / 获取日期时间间隔 ：
```
public static void durationDateTime() {
        LocalDateTime date3 = LocalDateTime.now();
        LocalDateTime date4 = LocalDateTime.of(2018, 1, 13, 22, 30, 10);
        Duration duration = Duration.between(date3, date4);
        System.out.println(date3 + " 与 " + date4 + " 间隔  " + "\n"
                + " 天 :" + duration.toDays() + "\n"
                + " 时 :" + duration.toHours() + "\n"
                + " 分 :" + duration.toMinutes() + "\n"
                + " 毫秒 :" + duration.toMillis() + "\n"
                + " 纳秒 :" + duration.toNanos() + "\n"
        );
    }
```
### 日期时间计算：
```
	public void addMonthDay() {
		LocalDateTime dt = LocalDateTime.of(2020, 1, 26, 20, 30, 59);
		System.out.println(dt);
		// 加5天减3小时:
		LocalDateTime dt2 = dt.plusDays(5).minusHours(3);
		System.out.println(dt2); // 2019-10-31T17:30:59
		// 减1月:
		LocalDateTime dt3 = dt2.minusMonths(1);
		System.out.println(dt3); // 2019-09-30T17:30:59
	}
```

## LocalTime
#### LocalTime获取时分秒：
```
        //获取小时
        int hour = localTime.getHour();
        int hour1 = localTime.get(ChronoField.HOUR_OF_DAY);
        //获取分
        int minute = localTime.getMinute();
        int minute1 = localTime.get(ChronoField.MINUTE_OF_HOUR);
        //获取秒
        int second = localTime.getMinute();
        int second1 = localTime.get(ChronoField.SECOND_OF_MINUTE);
```

### 参考资料：
https://www.liaoxuefeng.com/wiki/1252599548343744/1303871087444002
https://blog.csdn.net/weixin_38405253/article/details/100765007
https://www.jianshu.com/p/048ee8580639
