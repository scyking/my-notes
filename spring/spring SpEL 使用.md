# spring SpEL 使用

---

[toc]

## 概述

> SpEL（Spring Expression Language），即Spring表达式语言，是比JSP的EL更强大的一种表达式语言。

## 用法

### `@Value`

> 如果修饰成员变量，是从Spring容器中按照SpEL表达式筛选修改数据后，赋值给所修饰的变量；如果修饰方法形参，则是过滤传进来的参数值。

```java
// @Value能修饰成员变量和方法形参
// #{}内就是表达式的内容
@Value("#{表达式}")
public String arg;
```

### `<bean>` 配置

```xml
<bean id="xxx" class="com.java.XXXXX.xx">
    <!-- 同@Value,#{}内是表达式的值，可放在property或constructor-arg内 -->
    <property name="arg" value="#{表达式}">
</bean>
```

### `Expression`

> 用法比较灵活，可以在代码中使用SpEL进行数据的过滤和修改。

```java
public class SpELTest {
 
    public static void main(String[] args) {
 
        //创建ExpressionParser解析表达式
        ExpressionParser parser = new SpelExpressionParser();
        //表达式放置
        Expression exp = parser.parseExpression("表达式");
        //执行表达式，默认容器是spring本身的容器：ApplicationContext
        Object value = exp.getValue();
        
        /**如果使用其他的容器，则用下面的方法*/
        //创建一个虚拟的容器EvaluationContext
        StandardEvaluationContext ctx = new StandardEvaluationContext();
        //向容器内添加bean
        BeanA beanA = new BeanA();
        ctx.setVariable("bean_id", beanA);
        
        //setRootObject并非必须；一个EvaluationContext只能有一个RootObject，引用它的属性时，可以不加前缀
        ctx.setRootObject(XXX);
        
        //getValue有参数ctx，从新的容器中根据SpEL表达式获取所需的值
        Object value = exp.getValue(ctx);
    }
}
```

## 语法

### 直接量表达式

```java
"#{'Hello World'}"
```

### 直接使用`java`代码`new`/`instance of`

```java
Expression exp = parser.parseExpression("new Spring('Hello World')");
```

### 使用 `T`(`Type`)

```java
parser.parseExpression("T(Integer).MAX_VALUE");
```

### 变量

> 获取容器内的变量，可以使用 `#bean_id` 来获取。有两个特殊的变量，可以直接使用。

- #this 使用当前正在计算的上下文
- #root 引用容器的root对象

```java
String result2 = parser.parseExpression("#root").getValue(ctx, String.class);  

String s = new String("abcdef");
ctx.setVariable("abc",s);
//取id为abc的bean，然后调用其中的substring方法
parser.parseExpression("#abc.substring(0,1)").getValue(ctx, String.class);
```

### 方法调用

```java
Method parseInt = Integer.class.getDeclaredMethod("parseInt", String.class);
// registerFunction和setVariable都可以注册自定义函数
ctx.registerFunction("parseInt", parseInt);
ctx.setVariable("parseInt2", parseInt);
String expression1 = "#parseInt('3') == #parseInt2('3')";
```

### 运算符表达式

> 可以直接放在SpEL中。

- 算数表达式，`1+2-3*4/2`
- 比较表达式，`1>2`
- 逻辑表达式，`2>1 and (!true or !false)`
- 赋值表达式，`#variableName=value`
- 三目表达式，`表达式1?表达式2:表达式3`
- 正则表达式，`'123' matches '\\d{3}'`

### Elvis运算符

> 是三目运算符的特殊写法，可以避免null报错的情况。

```java
name != null? name : "other"
//简写为：
name?:"other"
```

### 安全保证

> 为了避免操作对象本身可能为null，取属性时报错，定义语法`对象?.变量|方法`。

```java
// 当对象为null时，直接返回“null”，不会抛出NullPointerException
list?.length
```

### 集合定义

> 使用`{表达式，……}`定义`List`，如 `{1,2,3}`。
> 对于字面量表达式列表，SpEL会使用`java.util.Collections.unmodifiableList`方法将列表设置为不可修改。

```java
List<Integer> result1 = parser.parseExpression("{1,2,3}").getValue(List.class);
```

### 集合访问

> SpEL目前支持所有集合类型和字典类型的元素访问。语法：`集合[索引]`、`map[key]`

```java
EvaluationContext context = new StandardEvaluationContext();

//即list.get(0)
int result1 = parser.parseExpression("{1,2,3}[0]").getValue(int.class);

//list获取某一项
Collection<Integer> collection = new HashSet<Integer>();
collection.add(1);
collection.add(2);

context.setVariable("collection", collection);
int result2 = parser.parseExpression("#collection[1]").getValue(context, int.class);

//map获取
Map<String, Integer> map = new HashMap<String, Integer>();
map.put("a", 1);

context.setVariable("map", map);
int result3 = parser.parseExpression("#map['a']").getValue(context, int.class);
```

### 集合修改

> 可以使用赋值表达式或`Expression`接口的`setValue`方法修改；

```java
// 赋值语句
int result = parser.parseExpression("#array[1] = 3").getValue(context, int.class);

// serValue方法
parser.parseExpression("#array[2]").setValue(context, 4);
```

### 集合选择

> 通过一定的规则对及格进行筛选，构造出另一个集合。语法，`(list|map).?[选择表达式]`
> 选择表达式结果必须是boolean类型，如果`true`则选择的元素将添加到新集合中，`false`将不添加到新集合中。

```java
// 选出数字大于2的值
parser.parseExpression("#collection.?[#this>2]").getValue(context, Collection.class);
```

### 集合投影

> 根据集合中的元素中通过选择来构造另一个集合，该集合和原集合具有相同数量的元素。语法，`SpEL使用“（list|map）.![投影表达式]`

```java
public class Book {
    public String name;             //书名
    public String author;           //作者
    public String publisher;        //出版社
    public double price;            //售价
    public boolean favorite;        //是否喜欢
}
```

```java
public class BookList {

    @Autowired
    protected ArrayList<Book> list = new ArrayList<Book>() ;
    
    protected int num = 0;
}
```

```java
// 从readList的list下筛选出favorite为true的子集合，再将他们的name字段投为新的list
@Value("#{list.?[favorite eq true].![name]}")
private ArrayList<String> favoriteBookName;
```
