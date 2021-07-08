# Spring 中设计模式

标签（空格分隔）： 设计模式 spring

---

[toc]

## 常用设计模式

|设计模式|举例|
|---|---|
|工厂模式|BeanFactory|
|单例模式|依赖注入bean，默认单例|
|适配器模式|HandlerAdapter|
|装饰者模式|xxxWrapper、xxxDecorator|
|代理模式|AOP底层实现|
|观察者模式|Listener实现|
|策略模式|Resource|
|模板模式|JDBC的抽象和对Hibernate的集成|

## 策略模式

### Resource接口实现

> 是具体资源访问策略的抽象，也是所有资源访问类所实现的接口。
> 接⼝本⾝没有提供访问任何底层资源的实现逻辑，针对不同的底层资源，Spring 将会提供不同的 `Resource` 实现类，不同的实现类负责不同的资源访问逻辑。

主要方法：

- `getInputStream()`：定位并打开资源，返回资源对应的输⼊流。每次调⽤都返回新的输⼊流。调⽤者必须负责关闭输⼊
流。
- `exists()`：返回 `Resource` 所指向的资源是否存在。
- `isOpen()`：返回资源⽂件是否打开，如果资源⽂件不能多次读取，每次读取结束应该显式关闭，以防⽌资源泄漏。
- `getDescription()`：返回资源的描述信息，通常⽤于资源处理出错时输出该信息，通常是全限定⽂件名或实际 URL。
- `getFile`：返回资源对应的 File 对象。
- `getURL`：返回资源对应的 URL 对象。

实现类：

- `UrlResource`：访问⽹络资源的实现类。
- `ClassPathResource`：访问类加载路径⾥资源的实现类。
- `FileSystemResource`：访问⽂件系统⾥资源的实现类。
- `ServletContextResource`：访问相对于 `ServletContext` 路径⾥的资源的实现类.
- `InputStreamResource`：访问输⼊流资源的实现类。
- `ByteArrayResource`：访问字节数组资源的实现类。

## 模板模式

> ⽗类定义了⻣架，某些特定⽅法由⼦类实现。

- 共同的⽅法：所有⼦类都会⽤到的代码
- 不同的⽅法：⼦类要覆盖的⽅法，分为两种：
    1. 抽象⽅法：⽗类中的是抽象⽅法，⼦类必须覆盖
    1. 钩⼦⽅法：⽗类中是⼀个空⽅法，⼦类继承了默认也是空的
