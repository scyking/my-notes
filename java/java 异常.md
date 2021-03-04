# java 异常

标签（空格分隔）： java

---

[toc]

## java异常简介
> Java异常是Java提供的一种识别及响应错误的一致性机制。Java异常机制用到的几个关键字：try、catch、finally、throw、throws。

- `try` 用于监听。将要被监听的代码(可能抛出异常的代码)放在try语句块之内，当try语句块内发生异常时，异常就被抛出。
- `catch` 用于捕获异常。catch用来捕获try语句块中发生的异常。
- `finally` finally语句块总是会被执行。它主要用于回收在try块里打开的物力资源(如数据库连接、网络连接和磁盘文件)。只有finally块，执行完成之后，才会回来执行try或者catch块中的return或者throw语句，如果finally中使用了return或者throw等终止方法的语句，则就不会跳回执行，直接停止。
- `throw` 用于抛出异常。
- `throws` 用在方法签名中，用于声明该方法可能抛出的异常。

## java异常架构
1. `Throwable`
是 Java 语言中所有错误或异常的超类。
包含两个子类: `Error` 和 `Exception`。它们通常用于指示发生了异常情况。
包含了其线程创建时线程执行堆栈的快照，它提供了`printStackTrace()`等接口用于获取堆栈跟踪数据等信息。
1. `Error`
和`Exception`一样，`Error`也是`Throwable`的子类。它用于指示合理的应用程序不应该试图捕获的严重问题，大多数这样的错误都是异常条件。和`RuntimeException`一样，编译器也不会检查`Error`。
1. `Exception`
`Exception`及其子类是 `Throwable` 的一种形式，它指出了合理的应用程序想要捕获的条件。
1. `RuntimeException`
是那些可能在 Java 虚拟机正常运行期间抛出的异常的超类。编译器不会检查`RuntimeException`异常。

## 异常分类

1. 运行时异常(Unchecked Exception)
- 定义: `RuntimeException`及其子类都被称为运行时异常。
- 特点: Java编译器不会检查它。也就是说，当程序中可能出现这类异常时，倘若既"没有通过throws声明抛出它"，也"没有用try-catch语句捕获它"，还是会编译通过。

1. 被检查的异常(Checked Exception)
- 定义: `Exception`类本身，以及`Exception`的子类中除了"运行时异常"之外的其它子类都属于被检查异常。
- 特点: Java编译器会检查它。此类异常，要么通过throws进行声明抛出，要么通过try-catch进行捕获处理，否则不能通过编译。被检查异常通常都是可以恢复的。

1. 错误(Unchecked Exception)
- 定义: `Error`类及其子类。
- 特点: 和运行时异常一样，编译器也不会对错误进行检查。当资源不足、约束失败、或是其它程序无法继续运行的条件发生时，就产生错误。程序本身无法修复这些错误的。例如，VirtualMachineError就属于错误。

## 常见异常

1. error
- VirtulMachineError
    1. `StackOverFlowError` 递归过深，递归没有出口。
    1. `OutOfMemoryError` JVM空间溢出，创建对象速度高于GC回收速度。
- AWTError

1. Exception
- IOException
    1. EOFException
    2. FileNotFoundException
- SQLException
- RuntimeException
    1. ArithmeticException
    2. MissingResourceException
    3. ClassNotFoundException
    4. NullPointerException
    5. IllegalArgumentException
    6. ArrayIndexOutOfBoundsException
    7. UnkownTypeException


    

 




