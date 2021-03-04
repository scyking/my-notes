# java8 方法引用

标签（空格分隔）：java8

---
[toc]

## 方法引用

> 方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

## 分类

|类型	|语法	|对应的Lambda表达式
|---
|类静态方法引用	|类名::staticMethod	|(args) -> 类名.staticMethod(args)
|实例化对象方法引用	|inst::instMethod	|(args) -> inst.instMethod(args)
|类方法引用	|类名::instMethod	|(inst,args) -> 类名.instMethod(args)
|类构建方法引用	|类名::new	|(args) -> new 类名(args)

## 举例

- 类构建方法引用
> 需要调用的构造器的参数列表要与函数式接口中抽象方法的参数列表保持一致。

    ```
    Function<String,String> fun1 = (a) -> new String(a);
    Function<String,String> fun2 = String::new;
    ```
- 类静态方法引用

    ```
    Function<Long, String> fun1 = (a) -> String.valueOf(a);
    Function<Long, String> fun2 = String::valueOf;
    ```
- 类方法引用
> 若Lambda参数列表中的第一个参数是实例方法的参数调用者，而第二个参数是实例方法的参数时，可以使用对象方法引用。

    ```
    BiPredicate<String, String> bp1 = (a, b) -> a.equals(b);
    BiPredicate<String, String> bq2 = String::equals;
    ```
- 实例化对象方法引用
> 调用已经存在的实例的方法，与静态方法引用不同的是类要先实例化，静态方法引用类无需实例化，直接用类名去调用。

    ```
    String str = "abc";
    Predicate<String> pd1 = (a) -> str.equals(a);
    Predicate<String> pd2 = str::equals;
    ```





