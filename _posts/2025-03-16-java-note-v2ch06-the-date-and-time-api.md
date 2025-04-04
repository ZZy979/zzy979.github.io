---
title: 《Java核心技术》笔记 卷II 第6章 日期和时间API
date: 2025-03-16 14:00:35 +0800
categories: [Java, Core Java]
tags: [java, date and time, formatting]
---
Java 1.0有一个`Date`类，事后证明它过于简单了。当Java 1.1引入`Calendar`类之后，`Date`类的大部分方法就被弃用了。但是`Calendar`的API并不出色，它的实例是可变的，并且没有处理闰秒之类的问题。第三次升级是Java 8中引入的`java.time` API，它弥补了过去的缺陷。在本章中，你将了解是什么使时间计算如此烦人，以及日期和时间API是如何解决这些问题的。

## 6.1 时间线
在Java中，`Instant`表示时间线上的一个点（时刻）。时间线的原点（称为**纪元**(epoch)）是本初子午线所处时区的1970年1月1日0点。从原点开始，时间按照每天86400秒向前或向后度量，精确到纳秒。最小值`Instant.MIN`为-1000000000年1月1日00:00，最大值`Instant.MAX`为1000000000年12月31日23:59:59。

静态方法`Instant.now()`返回当前时刻。可以用`equals()`和`compareTo()`方法来比较两个`Instant`，因此可以将其用作时间戳(timestamp)。

注：
* `Instant`对象内部存储了（UTC时间）距离纪元经过的秒数以及当前秒内的纳秒数，`getEpochSecond()`和`getNano()`方法分别返回这两个值。静态方法`ofEpochSecond()`创建给定时间戳（秒数）对应的`Instant`对象。
* 静态方法`System.currentTimeMillis()`返回当前时刻（单位为毫秒），等价于`Instant.now().toEpochMilli()`。
* `Instant` **不包含时区**。例如，时间戳3600表示UTC时间1970-1-1 01:00:00或北京时间1970-1-1 09:00:00。

为了得到两个时刻之间的差，可以使用静态方法`Duration.between()`。例如，下面测量算法的运行时间：

```java
Instant start = Instant.now();
runAlgorithm();
Instant end = Instant.now();
Duration timeElapsed = Duration.between(start, end);
long millis = timeElapsed.toMillis();
```

`Duration`表示时间间隔（持续时间），例如“34.5秒”。可以通过调用`toNanos()`、`toMillis()`、`toSeconds()`、`toMinutes()`、`toHours()`或`toDays()`来获得按不同单位度量的时间间隔长度。例如：

```java
Duration d = Duration.ofMillis(34500); // 34500 ms
long seconds = d.toSeconds(); // 34 s
long nanos = d.toNanos(); // 34500000000 ns
```

与`Instant`类似，`Duration`对象用一个`long`来存储秒数，一个额外的`int`存储纳秒数。如果需要纳秒级精度(`toNanos()`)，要当心溢出问题。一个`long`值可以存储大约300年对应的纳秒数。

`Duration` API包含许多用于执行算术运算的方法。例如，如果想检查一个算法是否至少比另一个算法快10倍，可以如下计算：

```java
Duration timeElapsed2 = Duration.between(start2, end2);
boolean overTenTimesFaster = timeElapsed.multipliedBy(10).minus(timeElapsed2).isNegative();
```

注释：`Instant`和`Duration`类都是不可变的，诸如`multipliedBy()`和`minus()`这样的方法都会返回一个新的实例。

程序清单6-1中的示例程序展示了如何使用`Instant`和`Duration`类来对算法计时。

[程序清单6-1 timeline/TimeLine.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch06/timeline/TimeLine.java)

## 6.2 本地日期
在Java API中有两种人类时间：**本地日期/时间**(local date/time)和**时区时间**(zoned time)。**本地日期/时间包含日期和/或当天的时间，但没有关联时区信息。** 例如，1903年6月14日是一个本地日期，它并不对应精确的时刻。相反，1969年7月16日09:32:00 EDT（阿波罗11号发射的时刻）是一个时区时间（EDT是美国东部夏令时间），表示时间线上一个精确的时刻。

由于夏令时(daylight savings time)等原因，API设计者建议不要使用时区时间，除非确实想表示绝对时刻。生日、假期、日程等通常最好表示成本地日期/时间。

`LocalDate`是带有年、月、日的本地日期（已经在4.2.2节介绍过）。要构造`LocalDate`对象，可以使用静态方法`now()`或`of()`：

```java
LocalDate today = LocalDate.now(); // Today's date
LocalDate alonzosBirthday = LocalDate.of(1903, 6, 14);
alonzosBirthday = LocalDate.of(1903, Month.JUNE, 14); // Uses the Month enumeration
```

在这里只需提供通常使用的年份和月份的数字，而不是像`java.util.Date`一样月份从0开始、年份从1900开始。或者也可以使用`Month`枚举。

`LocalDate`对象的方法见API文档。

例如，程序员日是每年的第256天。如下可以很容易地计算2014年的程序员日：

```java
LocalDate programmersDay = LocalDate.of(2014, 1, 1).plusDays(255); // 2014-09-13
```

注：也可以直接调用`LocalDate.ofYearDay(2014, 256)`。

两个时刻之间的差是`Duration`，而两个日期之间的差是`Period`，表示经过的年、月或日的数量。`LocalDate`类的`until()`方法返回两个本地日期之间的差（等价于`Period.between()`）。例如：

```java
var independenceDay = LocalDate.of(1776, 7, 4);
var christmas = LocalDate.of(1776, 12, 25);
Period period = independenceDay.until(christmas); // 5 months 21 days
long days = independenceDay.until(christmas, ChronoUnit.DAYS); // 174 days
```

警告：`LocalDate` API中的有些方法可能会创建出不存在的日期。例如，1月31日加上1个月不应该产生2月31日。这些方法会返回该月有效的最后一天，而不是抛出异常。例如，`LocalDate.of(2016, 1, 31).plusMonths(1)`和`LocalDate.of(2016, 3, 31).minusMonths(1)`都产生2016年2月29日。

`getDayOfWeek()`返回该日期是星期几，类型为`DayOfWeek`枚举。`DayOfWeek.MONDAY`的值为1，`DayOfWeek.SUNDAY`的值为7，`getValue()`返回对应的数值。`DayOfWeek`枚举的`plus()`和`minus()`方法计算加或减指定天数后是星期几。例如，`DayOfWeek.SATURDAY.plus(3)`结果为`DayOfWeek.TUESDAY`。

Java 9添加了两个有用的方法`datesUntil()`，生成当前到给定日期之间（可以指定步长）的`LocalDate`对象流。

```java
LocalDate start = LocalDate.of(2000, 1, 1);
LocalDate endExclusive = LocalDate.now();
Stream<LocalDate> allDays = start.datesUntil(endExclusive);
Stream<LocalDate> firstDaysInMonth = start.datesUntil(endExclusive, Period.ofMonths(1));
```

除了`LocalDate`之外，还有可以描述部分日期的`MonthDay`、`YearMonth`和`Year`类。例如，12月25日（没有指定年份）可以表示为一个`MonthDay`。

程序清单6-2中的示例程序展示了如何使用`LocalDate`类。

[程序清单6-2 localdates/LocalDates.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch06/localdates/LocalDates.java)

## 6.3 日期调整器
`LocalDate`是不可变的，但可以使用`withYear()`、`withMonth()`或`withDayOfMonth()`方法获得调整年、月或日之后的副本。例如：

```java
var d = LocalDate.of(2016, 1, 31);
var d2 = d.withMonth(2); // 2016-02-29
var d3 = d2.withYear(2018); // 2018-02-28
```

或者，也可以使用`with()`方法，该方法接受一个实现了`TemporalAdjuster`接口的对象。`date.with(adjuster)`等价于`adjuster.adjustInto(date)`。

对于日程安排应用来说，经常需要计算诸如“每个月第一个星期二”这样的日期。`TemporalAdjusters`类提供了许多用于常见调整的静态方法。例如，可以如下计算2016年4月的第一个星期二：

```java
LocalDate firstTuesday = LocalDate.of(2016, 4, 1).with(
    TemporalAdjusters.nextOrSame(DayOfWeek.TUESDAY)); // 2016-04-05
```

也可以通过实现`TemporalAdjuster`接口来创建自己的调整器。例如，下面是用于计算下一个工作日的调整器：

```java
TemporalAdjuster NEXT_WORKDAY = w -> {
    var result = (LocalDate) w;
    do {
        result = result.plusDays(1);
    } while (result.getDayOfWeek().getValue() >= 6);
    return result;
};
LocalDate backToWork = today.with(NEXT_WORKDAY);
```

注意，lambda表达式的参数类型（即该接口抽象方法的参数类型）为`Temporal`，必须强制转换为`LocalDate`。可以用`TemporalAdjusters.ofDateAdjuster()`方法来避免这种转换，该方法接受一个`UnaryOperator<LocalDate>`。

```java
TemporalAdjuster NEXT_WORKDAY = TemporalAdjusters.ofDateAdjuster(w -> {
    LocalDate result = w; // No cast
    do {
        result = result.plusDays(1);
    } while (result.getDayOfWeek().getValue() >= 6);
    return result;
});
```

## 6.4 本地时间
`LocalTime`表示一天中的时间，例如15:30:00。可以用`now()`或`of()`方法创建其实例：

```java
LocalTime rightNow = LocalTime.now();
LocalTime bedtime = LocalTime.of(22, 30); // or LocalTime.of(22, 30, 0)
```

`plus()`和`minus()`操作是按照24小时循环的。例如：

```java
LocalTime wakeup = bedtime.plusHours(8); // wakeup is 6:30:00
```

还有一个`LocalDateTime`类表示**日期和时间，没有关联时区**。这个类适合存储固定时区的时间点，例如用于课程或日程安排。但是，如果计算需要跨越夏令时，或者需要处理不同时区的用户，就应该使用接下来要讨论的`ZonedDateTime`类。

## 6.5 时区时间
互联网编号分配机构(Internet Assigned Numbers Authority, IANA)保存着一个全世界所有时区的数据库(<https://www.iana.org/time-zones>)。Java使用IANA数据库。

每个时区都有一个ID，例如America/New_York和Europe/Berlin。要获得所有可用的时区，调用`ZoneId.getAvailableZoneIds()`。

给定一个时区ID，静态方法`ZoneId.of()`产生对应的`ZoneId`对象。可以通过调用`LocalDateTime`类的`atZone()`方法将其转换为`ZonedDateTime`对象。或者可以通过调用静态方法`ZonedDateTime.of()`来构造对象。例如：

```java
ZonedDateTime apollo11launch = ZonedDateTime.of(1969, 7, 16, 9, 32, 0, 0, ZoneId.of("America/New_York"));
    // 1969-07-16T09:32-04:00[America/New_York]
```

`ZonedDateTime`表示一个具体的时刻。调用`toInstant()`方法可以得到对应的`Instant`对象（UTC时间的相同时刻）。例如：

```java
LocalDateTime epoch = LocalDateTime.of(1970, 1, 1, 0, 0, 0);
ZonedDateTime utcTime = ZonedDateTime.of(epoch, ZoneId.of("UTC")); // 1970-01-01T00:00Z[UTC]
ZonedDateTime shTime = ZonedDateTime.of(epoch, ZoneId.of("Asia/Shanghai")); // 1970-01-01T00:00+08:00[Asia/Shanghai]
long utcInstant = utcTime.toInstant(); // 1970-01-01T00:00:00Z
long shInstant = shTime.toInstant(); // 1969-12-31T16:00:00Z
```

反过来，调用`Instant`类的`atZone()`方法可以得到指定时区的`ZonedDateTime`对象（相同本地时间，可能不是同一时刻）。例如：

```java
Instant epoch = Instant.ofEpochSecond(0);
ZonedDateTime utcTime = epoch.atZone(ZoneId.of("UTC")); // 1970-01-01T00:00Z[UTC]
ZonedDateTime shTime = epoch.atZone(ZoneId.of("Asia/Shanghai")); // 1970-01-01T00:00+08:00[Asia/Shanghai]
long utcTimestamp = utcTime.toEpochSecond(); // 0
long shTimestamp = shTime.toEpochSecond(); // -28800
```

![Instant与ZonedDateTime之间的转换](/assets/images/java-note-v2ch06-the-date-and-time-api/Instant与ZonedDateTime之间的转换.png)

注释：UTC代表“协调世界时”，这是英文 "Coordinated Universal Time" 和法文 "Temps Universel Coordiné" 首字母缩写的折中。UTC是没有夏令时的格林威治皇家天文台时间。

`ZonedDateTime`的很多方法都与`LocalDateTime`相同（参见API文档），它们大多数都很简单，但是夏令时带来了一些复杂性。

当夏令时开始时，时钟会提前一小时。例如，2013年，中欧地区在3月31日2:00切换到夏令时。如果试图构造不存在的时间3月31日2:30，实际上会得到3:30。

```java
ZonedDateTime skipped = ZonedDateTime.of(
    LocalDate.of(2013, 3, 31),
    LocalTime.of(2, 30),
    ZoneId.of("Europe/Berlin"));
    // Constructs March 31 3:30
```

反过来，当夏令时结束时，时钟会后退一小时。这样就会有两个时刻具有相同的本地时间！如果构造这一段内的时间，会得到二者中较早的一个。

```java
ZonedDateTime ambiguous = ZonedDateTime.of(
    LocalDate.of(2013, 10, 27), // End of daylight savings time
    LocalTime.of(2, 30),
    ZoneId.of("Europe/Berlin"));
    // 2013-10-27T02:30+02:00[Europe/Berlin]
ZonedDateTime anHourLater = ambiguous.plusHours(1);
    // 2013-10-27T02:30+01:00[Europe/Berlin]
```

一个小时后的时间具有相同的小时和分钟数，但是时区偏移量发生了变化。

在调整跨越夏令时边界的日期时也需要注意。例如，如果要设置下周的会议，不要直接加上一个7天的`Duration`：

```java
ZonedDateTime nextMeeting = meeting.plus(Duration.ofDays(7));
    // Caution! Won't work with daylight savings time
```

而应该使用`Period`类：

```java
ZonedDateTime nextMeeting = meeting.plus(Period.ofDays(7)); // OK
```

程序清单6-3中的示例程序演示了`ZonedDateTime`类的用法。

[程序清单6-3 zonedtimes/ZonedTimes.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch06/zonedtimes/ZonedTimes.java)

### 小结
关键的日期时间类之间的转换如下图所示。

![日期时间类之间的转换](/assets/images/java-note-v2ch06-the-date-and-time-api/日期时间类之间的转换.png)

## 6.6 格式化和解析
`DateTimeFormatter`类提供了三种用于打印日期/时间的格式器：
* 预定义的标准格式器。
* 区域设置(locale)特定的格式器。
* 自定义模式的格式器。

标准格式器定义为静态常量。调用`format()`方法将日期/时间值格式化为字符串：

```java
DateTimeFormatter formatter = DateTimeFormatter.ISO_OFFSET_DATE_TIME;
String formatted = formatter.format(apollo11launch);
    // 1969-07-16T09:32:00-04:00
```

注：标准格式器`ISO_OFFSET_DATE_TIME`的含义为“年-月-日T时:分:秒-时区偏移量”。

要创建locale特定的格式器，使用静态方法`ofLocalized(Date|Time|DateTime)`并提供`FormatStyle`枚举值。例如：

```java
DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG);
String formatted = formatter.format(apollo11launch);
    // July 16, 1969 at 9:32:00 AM EDT
```

这些方法使用默认的locale。要改为不同的locale，只需使用`withLocale()`方法。

```java
formatted = formatter.withLocale(Locale.FRENCH).format(apollo11launch);
    // 16 juillet 1969 à 09:32:00 EDT
formatted = formatter.withLocale(Locale.CHINA).format(apollo11launch);
    // 1969年7月16日 EDT 上午9:32:00
```

`DayOfWeek`和`Month`枚举都有`getDisplayName()`方法，用于以不同的区域和格式给出星期和月份的名字。

```java
for (DayOfWeek w : DayOfWeek.values())
    System.out.print(w.getDisplayName(TextStyle.SHORT, Locale.ENGLISH) + " ");
    // Prints Mon Tue Wed Thu Fri Sat Sun
```

有关locale的更多信息参见第7章。

注释：`DateTimeFormatter`旨在替代`java.text.DateFormat`。如果为了向后兼容性需要后者的实例，可以调用`formatter.toFormat()`。

注：新的`DateTimeFormatter`是不可变、线程安全的，但不可序列化；旧的`DateFormat`可序列化，但不是线程安全的。

最后，可以通过`ofPattern()`方法指定模式来自定义日期时间格式。例如：

```java
formatter = DateTimeFormatter.ofPattern("E yyyy-MM-dd HH:mm");
```

会以 "Wed 1969-07-16 09:32" 的形式格式化日期时间。每个字母表示一个不同的时间字段，字母的重复次数选择特定格式。最常用的格式化符号如下表所示（完整列表参见API文档）。

| 符号 | 含义 | 示例 |
| --- | --- | --- |
| `yy` | 年（两位数） | 69 |
| `yyyy` | 年 | 1969 |
| `MM` | 月 | 07 |
| `MMM` | 月（简称） | Jul |
| `MMMM` | 月（完整名称） | July |
| `dd` | 日 | 16 |
| `HH` | 小时 | 09 |
| `mm` | 分钟 | 32 |
| `ss` | 秒 | 00 |

要从字符串解析日期/时间值，使用日期/时间类的静态方法`parse()`或`DateTimeFormatter`类的`parse()`方法。例如：

```java
LocalDate churchsBirthday = LocalDate.parse("1903-06-14");
ZonedDateTime apollo11launch = ZonedDateTime.parse(
    "1969-07-16 03:32:00-0400", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ssxx"));
```

第一个调用使用标准格式器`ISO_LOCAL_DATE`（`LocalDate`的默认格式），第二个使用自定义格式器。

程序清单6-4中的程序展示了如何格式化和解析日期与时间。

[程序清单6-4 formatting/Formatting.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch06/formatting/Formatting.java)

## 6.7 与遗留代码互操作
作为全新的创造，`java.time` API必须能够与已有的类进行互操作，特别是无处不在的`java.util.Date`、`java.util.GregorianCalendar`和`java.sql.Date/Time/Timestamp`。下图总结了这些转换。

![java.time类与遗留类之间的转换](/assets/images/java-note-v2ch06-the-date-and-time-api/java.time类与遗留类之间的转换.png)
