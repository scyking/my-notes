# java8 lambda 表达式

标签（空格分隔）： java8

---

[toc]

## 函数式接口
> 是只显式声明一个抽象方法的接口。为保证方法数量不多不少，java8提供了一个专用注解`@FunctionalInterface`，这样，当接口中声明的抽象方法多于或少于一个时就会报错。

## lambda 表达式(`(参数列表) -> 表达式`)
java代码:
```
public int add(int x, int y) {
    return x + y;
}
```
转成Lambda表达式：
```
(int x, int y) -> x + y;
```
参数类型也可以省略，Java编译器会根据上下文推断出来：
```
(x, y) -> x + y; //返回两数之和
// 或者
(x, y) -> { return x + y; } //显式指明返回值
```

## Lambda表达式和函数式接口结合

- 新建无参函数式接口（先演示无参）；
```
@FunctionalInterface
public interface InterfaceWithNoParam {
    void run();
}
```
- 新建包含属性为函数式接口的类；
```
public class TestJava8{
    InterfaceWithNoParam param;
}
```
- 实现函数式接口；
```
public class TestJava8{
	//匿名内部类
	InterfaceWithNoParam param1 = new InterfaceWithNoParam() {
        @Override
        public void run() {
            System.out.println("通过匿名内部类实现run()");
        }
    };
	//Lambda表达式
    //空括号表示无参
	InterfaceWithNoParam param = () -> System.out.println("通过Lambda表达式实现run()") ;
}
```
- 测试函数式接口的方法；
```
@Test
public void testIntfaceWithNoparam() {
    this.param.run();
    this.param1.run();
}
```
运行结果
```
通过Lambda表达式实现run()
通过匿名内部类实现run()
```

## 其他形式的函数式接口及实现
- 有参无返回值
- 无参有返回值
- 有参有返回值
