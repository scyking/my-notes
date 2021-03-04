# java8 stream

标签（空格分隔）：java8

---

[toc]

## 介绍

元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。
```
+--------------------+       +------+   +------+   +---+   +-------+
| stream of elements +-----> |filter+-> |sorted+-> |map+-> |collect|
+--------------------+       +------+   +------+   +---+   +-------+
```
以上的流程转换为 Java 代码为：
```
List<Integer> transactionsIds = 
widgets.stream()
             .filter(b -> b.getColor() == RED)
             .sorted((x,y) -> x.getWeight() - y.getWeight())
             .mapToInt(Widget::getWeight)
             .sum();
```             

## 创建流方式

- `Arrays.stream`(有限流，长度即对象长度)
- `Stream.of`(有限流)
- `Collection.stream`(有限流)
- `Stream.iterate`(无限流，最大是`Long.MAX_VALUE`)
- `Stream.generate`(无限流)

## 主要方法

### forEach
> Stream 提供了新的方法 'forEach' 来迭代流中的每个数据。以下代码片段使用 forEach 输出了10个随机数：

```
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
```

### map
> `<R> Stream<R> map(Function<? super T, ? extends R> mapper);`
> 用于映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数：

```
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
// 获取对应的平方数
List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
```

### filter
> `Stream<T> filter(Predicate<? super T> predicate);`
> 用于通过设置的条件过滤出元素。以下代码片段使用 filter 方法过滤出空字符串：

```
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
long count = strings.stream().filter(string -> string.isEmpty()).count();
```

### flatMap
> `<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);`
> 用于把两个流变成一个流返回

```
String[] strs = { "aaa", "bbb", "ccc" };
Arrays.stream(strs).map(str -> str.split("")).forEach(System.out::println);// Ljava.lang.String;@53d8d10a
Arrays.stream(strs).map(str -> str.split("")).flatMap(Arrays::stream).forEach(System.out::println);// aaabbbccc
Arrays.stream(strs).map(str -> str.split("")).flatMap(str -> Arrays.stream(str)).forEach(System.out::println);// aaabbbccc
```

### limit，skip
> 截断--取先maxSize个对象：`Stream<T> limit(long maxSize);`
> 截断--忽略前N个对象：`Stream<T> skip(long n)`
> limit 方法用于获取指定数量的流。 以下代码片段：

```
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
```

### sorted
> 排序：`Stream<T> sorted();`
> 根据属性排序：`Stream<T> sorted(Comparator<? super T> comparator);`
> sorted 方法用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序：

```
Random random = new Random();
random.ints().limit(10).sorted().forEach(System.out::println);
```

### 并行（parallel）程序
> parallelStream 是流并行处理程序的代替方法。

```
// 输出空字符串的数量
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
long count = strings.parallelStream().filter(string -> string.isEmpty()).count();
```

### Collectors，collect(收集器操作)
> <R> R collect(Supplier<R> supplier,BiConsumer<R, ? super T> accumulator,BiConsumer<R, R> combiner);
> <R, A> R collect(Collector<? super T, A, R> collector);
> Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串：

```
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
 
System.out.println("筛选列表: " + filtered);
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
System.out.println("合并字符串: " + mergedString);
```

### 统计结果的收集器
> 主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。

```
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
 
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();
 
System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin());
System.out.println("所有数之和 : " + stats.getSum());
System.out.println("平均数 : " + stats.getAverage());
```

## 其他中间操作

```
//去重复
Stream<T> distinct();
//对对象进行操作，注意与map区别
Stream<T> peek(Consumer<? super T> action);
```

## 其他终端操作

```
// 对集合流进行遍历操作，与forEach相比，在使用并行流`parallelStream`时，会按照顺序取数据
void forEachOrdered(Consumer<? super T> action);

// 返回数组
Object[] toArray();
<A> A[] toArray(IntFunction<A[]> generator);

// 获取最小值
Optional<T> min(Comparator<? super T> comparator);  
// 获取最大值  
Optional<T> max(Comparator<? super T> comparator);
// 获取集合第一个对象
Optional<T> findFirst();  
// 获取集合中任意对象  
Optional<T> findAny();

// 获取元素和
long count();  
// 判断的条件里，任意一个元素成功，返回true
boolean anyMatch(Predicate<? super T> predicate);  
// 判断条件里的元素，所有的都是，返回true
boolean allMatch(Predicate<? super T> predicate);  
// 与allMatch相反，判断条件里的元素，所有的都不是，返回true
boolean noneMatch(Predicate<? super T> predicate);

// 归约操作，将流归约成一个值的操作叫做归约操作。用函数式编程语言的术语来说，这种称为折叠（fold）；
T reduce(T identity, BinaryOperator<T> accumulator);
Optional<T> reduce(BinaryOperator<T> accumulator);
<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner);
```

