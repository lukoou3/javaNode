## Date/Time API
最开始java 1.0时代,开始使用Date时间类,相当难用,他是以1900年起始,月份是从0开始的,如果你要表示2018年11月25日,那么就要这样
```java
Date date = new Date(118,10,25);
System.out.println(date.toLocaleString()); //2018-11-25 0:00:00
```

之后Date大多数方法已经废弃,却而代之的是Calendar类,但是这个类的月份依旧是从0开始的,但是年份不是从1900开始的,但是解析日期的DateFormat方法只在Date中有,并且他不是线程安全的

Date类名为Date，却表示Date和Time两部分

Date和Calendar都是可变的,这将带来很多麻烦,java8中在java.time中整合了很多Joda-Time的特性,Joda-Time之前是第三方类库

总结：
```java
Instant：时间戳
Duration：持续时间，时间差
LocalDate：只包含日期，比如：2016-10-20
LocalTime：只包含时间，比如：23:12:10
LocalDateTime：包含日期和时间，比如：2016-10-20 23:14:21
Period：时间段
ZoneOffset：时区偏移量，比如：+8:00
ZonedDateTime/OffsetDateTime：带时区的时间
```

### 使用LocalDate和LocalTime
从类名就可以看出,LocalDate是表示年月日,而LocalTime是表示时分秒

#### LocalDate使用
Java8用LocalDate取代Date，原因是Date实在是太难用了。

* Date月份从0开始，一月是0，十二月是11，LocalDate月份和星期都改成了enum。    
* Date和SimpleDateFormatter都不是线程安全的，而LocalDate和LocalTime和最基本的String一样，是不变类型，不但线程安全，而且不能修改。    
* Date是一个“万能接口”，它包含日期、时间，还有毫秒数。在新的Java 8中，日期和时间被明确划分为LocalDate和LocalTime，当然，LocalDateTime才能同时包含日期和时间。  

取当前日期：
```java
LocalDate today = LocalDate.now(); 
```
根据年月日取日期：
```java
LocalDate crischristmas = LocalDate.of(2018, 12, 25);
```
根据字符串取日期：
```java
LocalDate endOfFeb = LocalDate.parse("2018-02-28"); 
```

Date和LocalDate互转:
```java
public static LocalDate dateToLocalDate(Date d) {
        Instant instant = d.toInstant();
        LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
        return localDateTime.toLocalDate();
    }

public static Date localDateToDate(LocalDate localDate) {
        Instant instant = localDate.atStartOfDay().atZone( ZoneId.systemDefault()).toInstant();
        return Date.from(instant);
    }
```

测试：
```java
//获取现在时间
LocalDate now = LocalDate.now();  //2018-11-25
LocalDate date = LocalDate.of(2018,11,25);
int year = date.getYear();  //2018
Month month = date.getMonth(); // NOVEMBER
int monthValue = date.getMonth().getValue();  //11
int dayOfMonth = date.getDayOfMonth(); //25
DayOfWeek dayOfWeek = date.getDayOfWeek();  //SUNDAY
int dayOfWeekValue = dayOfWeek.getValue(); //7
int i = date.lengthOfMonth();   //30
boolean leapYear = date.isLeapYear();  //闰年false
```

除了getxxx方法,还可以使用TemporalField参数给get方法获取相同的信息
```java
LocalDate localDate = LocalDate.of(2018, 11, 25);
int year = localDate.get(ChronoField.YEAR);
int month = localDate.get(ChronoField.MONTH_OF_YEAR);
int dayofweek = localDate.get(ChronoField.DAY_OF_WEEK);
```
如上的ChronoField是TemporalField的实现

#### LocalTime的使用
LocalTime包含毫秒：
```java
LocalTime now = LocalTime.now(); // 11:09:09.240
```
清除毫秒：
```java
LocalTime now = LocalTime.now().withNano(0)); // 11:09:09
```
构造时间：
```java
LocalTime zero = LocalTime.of(0, 0, 0); // 00:00:00

//时间也是按照ISO格式识别，但可以识别以下3种格式：
//12:00
//12:01:02
//12:01:02.345
LocalTime mid = LocalTime.parse("12:00:00"); // 12:00:00
```
Date和LocalTime互转:
```java
public static LocalTime dateToLocalTime(Date d) {
    Instant instant = d.toInstant();
    LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
    return localDateTime.toLocalTime();
}
```

测试：
```java
LocalTime time = LocalTime.of(8,34,00);
int hour = time.getHour();  //8
int minute = time.getMinute();  //34
int second = time.getSecond();  //0
int nano = time.getNano();  //0
```

#### LocalDate和LocalTime的解析
```java
LocalTime time = LocalTime.parse("08:24");
LocalDate date = LocalDate.parse("2018-11-25");
```
解析不了会出DateTimeParseException

### LocalDateTime(合并日期和时间)
LocalDateTime类是LocalDate和LocalTime的结合体，可以通过of()方法直接创建，也可以调用LocalDate的atTime()方法或LocalTime的atDate()方法将LocalDate或LocalTime合并成一个LocalDateTime：
```java
LocalDateTime ldt1 = LocalDateTime.of(2017, Month.JANUARY, 4, 17, 23, 52);
LocalDate localDate = LocalDate.of(2017, Month.JANUARY, 4);
LocalTime localTime = LocalTime.of(17, 23, 52);
LocalDateTime ldt2 = localDate.atTime(localTime);
```

LocalDateTime也提供用于向LocalDate和LocalTime的转化：
```java
LocalDate date = ldt1.toLocalDate();
LocalTime time = ldt1.toLocalTime();
```

Date和LocalDateTime互转:
```java
public static Date localDateTimeToDate(LocalDateTime localDateTime) {
    Instant instant = localDateTime.atZone(ZoneId.systemDefault()).toInstant();
    return Date.from(instant);
}

public static LocalDateTime dateToLocalDateTime(Date d) {
    Instant instant = d.toInstant();
    LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
    return localDateTime;
}
```

### LocalDateTime与框架兼容性
myBatis (mybatis-spring v1.3.0) 暂不支持 localDateTime

Json (com.fasterxml.jackson.core v2.5.4) 貌似也没特别的支持

最新JDBC映射将把数据库的日期类型和Java 8的新类型关联起来：
```java
SQL -> Java
--------------------------
date -> LocalDate
time -> LocalTime
timestamp -> LocalDateTime
```


所以, 在与 数据库 交互, 与 Json 交互时还是建议使用 java.util.Date

总结:
当前情况下, 业务计算建议使用 LocalDateTime, 业务计算以外建议转化为 Date

### 日期的操作(增加和减少日期)
**以上这些日期-时间对象都是不可修改的，为了更好地支持函数式编程，确保线程安全。**

若需要修改个LocalDate对象,withAttribute方法会创建对象的一个副本，并按照需要修改它的属性。(with方法也可以达到同样目的，它接受的第一个参数是一个TemporalField对象)
```java
//所有的方法都返回一个修改了属性的对象。它们都不会修改原来的对象！
LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.withYear(2011);
LocalDate date3 = date2.withDayOfMonth(25);
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 9);
```

以相对方式修改localDate属性
```java
LocalDate date1 = LocalDate.of(2014, 3, 18);
LocalDate date2 = date1.plusWeeks(1);
LocalDate date3 = date2.minusYears(3);
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS); //2011-09-25
```
plus方法也是通用方法，它和minus方法都声明于Temporal接口中,通过这些方法，对TemporalUnit对象加上或者减去一个数字，能非常方便地将Temporal对象前溯或者回滚至某个时间段，通过ChronoUnit枚举可以非常方便地实现TemporalUnit接口。


测试：
```java
LocalDate date = LocalDate.of(2017, 1, 5);          // 2017-01-05

LocalDate date1 = date.withYear(2016);              // 修改为 2016-01-05
LocalDate date2 = date.withMonth(2);                // 修改为 2017-02-05
LocalDate date3 = date.withDayOfMonth(1);           // 修改为 2017-01-01

LocalDate date4 = date.plusYears(1);                // 增加一年 2018-01-05
LocalDate date5 = date.minusMonths(2);              // 减少两个月 2016-11-05
LocalDate date6 = date.plus(5, ChronoUnit.DAYS);    // 增加5天 2017-01-10

LocalDate date7 = date.with(nextOrSame(DayOfWeek.SUNDAY));      // 返回下一个距离当前时间最近的星期日
LocalDate date9 = date.with(lastInMonth(DayOfWeek.SATURDAY));   // 返回本月最后一个星期六
```

```
dayOfWeekInMonth 返回同一个月中每周的第几天
firstDayOfMonth 返回当月的第一天
firstDayOfNextMonth 返回下月的第一天
firstDayOfNextYear 返回下一年的第一天
firstDayOfYear 返回本年的第一天
firstInMonth 返回同一个月中第一个星期几
lastDayOfMonth 返回当月的最后一天
lastDayOfNextMonth 返回下月的最后一天
lastDayOfNextYear 返回下一年的最后一天
lastDayOfYear 返回本年的最后一天
lastInMonth 返回同一个月中最后一个星期几
next / previous 返回后一个/前一个给定的星期几
nextOrSame / previousOrSame 返回后一个/前一个给定的星期几，如果这个值满足条件，直接返回
```

### 格式化日期
新的日期API中提供了一个DateTimeFormatter类用于处理日期格式化操作，与之前的DateFormat相比,DateTimeFormatter是线程安全的。它被包含在java.time.format包中，Java 8的日期类有一个format()方法用于将日期格式化为字符串，该方法接收一个DateTimeFormatter类型参数：
```java
LocalDateTime dateTime = LocalDateTime.now();
String strDate1 = dateTime.format(DateTimeFormatter.BASIC_ISO_DATE);    // 20170105
String strDate2 = dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE);    // 2017-01-05
String strDate3 = dateTime.format(DateTimeFormatter.ISO_LOCAL_TIME);    // 14:20:16.998
String strDate4 = dateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));   // 2017-01-05
String strDate5 = dateTime.format(DateTimeFormatter.ofPattern("今天是：YYYY年 MMMM DD日 E", Locale.CHINESE)); // 今天是：2017年 一月 05日 星期四
```
同样，日期类也支持将一个字符串解析成一个日期对象，例如：
```java
String strDate6 = "2017-01-05";
String strDate7 = "2017-01-05 12:30:05";

LocalDate date = LocalDate.parse(strDate6, DateTimeFormatter.ofPattern("yyyy-MM-dd"));
LocalDateTime dateTime1 = LocalDateTime.parse(strDate7, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```

### Instant
Instant用于表示一个时间戳，可以精确到纳秒（Nano-Second）。

可以通过向静态工厂方法ofEpochSecond传递一个代表秒数的值创建一个该类的实例。静态工厂方法ofEpochSecond还有一个增强的重载版本，它接收第二个以纳秒为单位的参数值，对传入作为秒数的参数进行调整。
```java
Instant.ofEpochSecond(3);//1970-01-01T00:00:03Z
Instant.ofEpochSecond(3, 0);
Instant.ofEpochSecond(2, 1_000_000_000);//2秒之后加上100万纳秒（1秒）
Instant.ofEpochSecond(4, -1_000_000_000); 4秒之前100弯纳秒（1秒）
```
由于Instant类也支持静态工厂方法now，它能够帮你获取当前时刻的时间戳
```java
Instant now = Instant.now();
```

### Duration和Period
Duration的内部实现与Instant类似，也是包含两部分：seconds表示秒，nanos表示纳秒。两者的区别是Instant用于表示一个时间戳（或者说是一个时间点），而Duration表示一个时间段。可以通过Duration.between()方法创建Duration对象：
```java
LocalDateTime from = LocalDateTime.of(2019, Month.JANUARY, 5, 10, 7, 0);    // 2017-01-05 10:07:00
LocalDateTime to = LocalDateTime.of(2019, Month.FEBRUARY, 5, 10, 7, 0);     // 2017-02-05 10:07:00
Duration duration = Duration.between(from, to);     // 表示从 2017-01-05 10:07:00 到 2017-02-05 10:07:00 这段时间

long days = duration.toDays();              // 这段时间的总天数
long hours = duration.toHours();            // 这段时间的小时数
long minutes = duration.toMinutes();        // 这段时间的分钟数
long seconds = duration.getSeconds();       // 这段时间的秒数
long milliSeconds = duration.toMillis();    // 这段时间的毫秒数
long nanoSeconds = duration.toNanos();      // 这段时间的纳秒数
```

Period在概念上和Duration类似，区别在于Period是以年月日来衡量一个时间段，比如2年3个月6天：
```java
Period period = Period.of(2, 3, 6);

// 2017-01-05 到 2017-02-05 这段时间
Period period = Period.between(
                LocalDate.of(2019, 1, 5),
                LocalDate.of(2019, 2, 5));
```





