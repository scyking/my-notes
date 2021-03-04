# java8 日期处理

标签（空格分隔）： java8

---

[toc]

## 与之前版本API比较

旧版api存在的问题：

- 非线程安全 − java.util.Date 是非线程安全的，所有的日期类都是可变的，这是Java日期类最大的问题之一。
- 设计很差 − Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
- 时区处理麻烦 − 日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。

java8 新增api（重点）：

- Local(本地) − 简化了日期时间的处理，没有时区的问题。
- Zoned(时区) − 通过制定的时区处理日期时间。

## java8 相关类

- Instant         时间戳
- Duration        持续时间、时间差
- LocalDate       只包含日期，比如：2018-09-24
- LocalTime       只包含时间，比如：10:32:10
- LocalDateTime   包含日期和时间，比如：2018-09-24 10:32:10
- Peroid          时间段
- ZoneOffset      时区偏移量，比如：+8:00
- ZonedDateTime   带时区的日期时间
- Clock           时钟，可用于获取当前时间戳
- java.time.format.DateTimeFormatter      时间格式化类

## 常用操作

### 日期操作

1. 获取当天日期
Java 8中的` LocalDate` 用于表示当天日期。和`java.util.Date`不同，它只有日期，不包含时间。
    ```
    public static void main(String[] args) {
    　　LocalDate date = LocalDate.now();
    　　System.out.println("当前日期=" + date);
    }
    ```

1. 构造指定日期
调用工厂方法LocalDate.of()创建任意日期。该方法需要传入年、月、日做参数，返回对应的LocalDate实例。这个方法的好处是没再犯老API的设计错误，比如年度起始于1900，月份是从0开始等等。
    ```
    public static void main(String[] args) {
        LocalDate date = LocalDate.of(2000, 1, 1);
        System.out.println("千禧年=" + date);
    }
    ```

1. 获取年月日信息

    ```
    public static void main(String[] args) {
        LocalDate date = LocalDate.now();
        System.out.printf("年=%d， 月=%d， 日=%d", date.getYear(), date.getMonthValue(), date.getDayOfMonth());
    }
    ``` 

1. 比较两个日期是否相等

    ```
    public static void main(String[] args) {
        LocalDate now = LocalDate.now();
        LocalDate date = LocalDate.of(2018, 9, 24);
        System.out.println("日期是否相等=" + now.equals(date));
    }
    ```

### 时间操作

1. 获取当前时间
Java 8中的 `LocalTime` 用于表示当天时间。和`java.util.Date`不同，它只有时间，不包含日期。

    ```
    public static void main(String[] args) {
        LocalTime time = LocalTime.now();
        System.out.println("当前时间=" + time);
    }
    ```
    
### 时区操作
1. 创建带有时区的日期时间
Java 8不仅分离了日期和时间，也把时区分离出来了。现在有一系列单独的类如ZoneId来处理特定时区，ZoneDateTime类来表示某时区下的时间。
    ```
    public static void main(String[] args) {
        
        // 上海时间
        ZoneId shanghaiZoneId = ZoneId.of("Asia/Shanghai");
        ZonedDateTime shanghaiZonedDateTime = ZonedDateTime.now(shanghaiZoneId);
        
        // 东京时间
        ZoneId tokyoZoneId = ZoneId.of("Asia/Tokyo");
        ZonedDateTime tokyoZonedDateTime = ZonedDateTime.now(tokyoZoneId);
        
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        System.out.println("上海时间: " + shanghaiZonedDateTime.format(formatter));
        System.out.println("东京时间: " + tokyoZonedDateTime.format(formatter));
        
    }
    ```
    
### 比较与计算

1. 日期时间计算
Java8提供了新的plusXxx()方法用于计算日期时间增量值，替代了原来的add()方法。新的API将返回一个全新的日期时间示例，需要使用新的对象进行接收。

    ```
        public static void main(String[] args) {
            
    　　　　 // 时间增量
            LocalTime time = LocalTime.now();
            LocalTime newTime = time.plusHours(2);
            System.out.println("newTime=" + newTime);
            
    　　　　　// 日期增量
            LocalDate date = LocalDate.now();
            LocalDate newDate = date.plus(1, ChronoUnit.WEEKS);
            System.out.println("newDate=" + newDate);
            
        }
    ```

1. 日期时间比较
Java8提供了isAfter()、isBefore()用于判断当前日期时间和指定日期时间的比较

    ```
    public static void main(String[] args) {
        
        LocalDate now = LocalDate.now();
        
        LocalDate date1 = LocalDate.of(2000, 1, 1);
        if (now.isAfter(date1)) {
            System.out.println("千禧年已经过去了");
        }
        
        LocalDate date2 = LocalDate.of(2020, 1, 1);
        if (now.isBefore(date2)) {
            System.out.println("2020年还未到来");
        }
        
    }
    ```
    
### 格式化
`SimpleDateFormat`线程不安全，java8 引入`DateTimeFormatter`。

1. 使用预定义格式解析与格式化日期

    ```
    public static void main(String[] args) {
        
        // 解析日期
        String dateText = "20180924";
        LocalDate date = LocalDate.parse(dateText, DateTimeFormatter.BASIC_ISO_DATE);
        System.out.println("格式化之后的日期=" + date);
        
        // 格式化日期
        dateText = date.format(DateTimeFormatter.ISO_DATE);
        System.out.println("dateText=" + dateText);
        
    }
    ```

1. 日期和字符串的相互转换

    ```
    public static void main(String[] args) {
        
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        
        // 日期时间转字符串
        LocalDateTime now = LocalDateTime.now();
        String nowText = now.format(formatter);
        System.out.println("nowText=" + nowText);
        
        // 字符串转日期时间
        String datetimeText = "1999-12-31 23:59:59";
        LocalDateTime datetime = LocalDateTime.parse(datetimeText, formatter);
        System.out.println(datetime);
        
    }
    ```
    

