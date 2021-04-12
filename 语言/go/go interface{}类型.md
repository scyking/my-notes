# go interface{}类型

标签：go

---

[toc]

## 空接口 interface{}

> `interface{}` 类型是没有方法的接口。由于没有 `implements` 关键字，所以所有类型都至少实现了 0 个方法，所以所有类型都实现了空接口。

> 类似 `java` 中 `Object`

## 接口

> 接口是一组方法签名(声明的是一组方法的集合)。

### 核心概念

> 根据类型可以执行的操作而不是其所能容纳的数据类型来设计抽象。

> 类型包含了接口中的方法，就可以称他实现了该接口。

1. 接口

    ```
    type Animal interface {
        Speak() string
    }
    ```

1. 实现

    ```
    type Dog struct {
    }
     
    func (d Dog) Speak() string {
        return "Woof!"
    }
     
    type Cat struct {
    }
     
    func (c Cat) Speak() string {
        return "Meow!"
    }
    ```
1. 调用
    ```
    animals := []Animal{Dog{}, Cat{}, Llama{}, JavaProgrammer{}}
    ```
## 指针和接口

> 接口定义没有规定一个实现者是否应该使用一个指针接收器或一个值接收器来实现接口。当给定一个接口值时，不能保证底层类型是否为指针。

1. 将 Cat 的 Speak() 方法改为指针接收器

    ```
    func (c *Cat) Speak() string {
        return "Meow!"
    }
    ```

1. 报错

    ```
    cannot use Cat literal (type Cat) as type Animal in array or slice literal:
        Cat does not implement Animal (Speak method has pointer receiver)
    ```

    可以通过传入一个指针 `new(Cat)` 或者 `&Cat{}` 来修复这个错误。

1. 原因
> 任何一个 `Cat` 类型的值可能会有很多 `*Cat` 类型的指针指向它，如果我们尝试通过 `Cat` 类型的值来调用 `*Cat` 的方法，根本就不知道对应的是哪个指针。
> 相反，如果 `Cat` 类型上有一个方法，通过 `*Cat` 来调用这个方法可以确切的找到该指针对应的 `Cat` 类型的值。

1. 总结
> 指针类型可以通过其相关的值类型来访问值类型的方法，但是反过来不行。
> 因为 Go 中的所有东西都是按值传递的。每次调用函数时，传入的数据都会被复制。对于具有值接收者的方法，在调用该方法时将复制该值。