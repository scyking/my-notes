# java8 函数式接口

标签（空格分隔）： java8

---

[toc]

## 概念
> 函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。
> 函数式接口可以被隐式转换为 lambda 表达式。

函数式接口：
```
@FunctionalInterface
interface GreetingService 
{
    void sayMessage(String message);
}
```
实现:
```
// Lambda
GreetingService greetService1 = message -> System.out.println("Hello " + message);

// java8 之前使用匿名类
GreetingService greetService2 = new GreetingService() {
            @Override
            public void sayMessage(String message) {
                System.out.println("Hello " + message);
            }
        };
```
## 主要类型

|接口名|说明
|---
|`Function<T,R>` |接收一个T类型的参数，返回一个R类型的结果
|`Consumer<T>` |接收一个T类型的参数，不返回值
|`Predicate<T>` |接收一个T类型的参数，返回一个boolean类型的结果
|`Supplier<T>` |不接受参数，返回一个T类型的结果

## 可接收两个参数函数式接口

|接口名|说明
|---
|`BiFunction<T, U, R>`|接收T类型和U类型的两个参数，返回一个R类型的结果
|`BiConsumer<T, U>`|接收T类型和U类型的两个参数，不返回值
|`BiPredicate<T, U>`|接收T类型和U类型的两个参数，返回一个boolean类型的结果




