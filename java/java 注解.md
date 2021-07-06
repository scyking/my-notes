# java 注解

标签（空格分隔）： java

---

[toc]

## 概念
> 注解是一种配置，信息和程序的分离。
> xml等配置文件，修改之后不需要重新编译 java 文件。但是，注解本身就在 java 类文件中，被 jvm 读取，并存放到内存中。因此，修改之后需要重新编译。

## 元注解
### `@Retention` 
> 描述注解的声明周期。

```
RetentionPolicy{ SOURCE  ,  CLASS  ,  RUNTIME }
```
- `SOURCE` ，源文件保留。表示这个 Annotation 类型的信息只会保留在程序源码里，源码如果经过了编译之后，Annotation 的数据就会消失，并不会保留在编译好的.class文件里面。  
- `CLASS` ，class 保留。表示这个 Annotation 类型的信息会保留在程序源码里，同时也会保留在编译好的 .class 文件里，在执行的时候，并不会把这些信息加载到 jvm 中去。未设定 Retention 的值时，默认为 CLASS。
- `RUNTIME` ，运行时保留。 表示在源码，编译好的 .class 文件中保留信息，在执行的时候会把这些信息加载到 jvm 中。 
### `@Target` 
> 描述注解的使用范围。
```
ElementType { TYPE  ,  FIELD  ,  METHOD  ,  PARAMETER  ,  CONSTRUCTOR , PACKAGE  ,  LOCAL_VARIABLE}
```
- `TYPE` : 用于描述类或者接口
- `FIELD` : 用于描述类的属性
- `METHOD` : 用于描述类的方法
- `PARAMETER` : 用于描述方法的参数
- `CONSTRUCTOR` : 用于描述构造器
- `LOCAL_VARIABLE` : 用于描述局部变量
- `PACKAGE` : 用于描述包
### `@Documented`
> 标记注解。
> 在自定义注解的时候，让这个Annotation类型的信息能够显示在java api说明文档上。

### `@Inherited`
> 标记注解。
> 在自定义注解中，可以将Annotation中的数据继承给子类。

## 自定义注解
> 使用`@interface`自定义注解时，自动继承`java.lang.annotation.Annotation`接口。方法的名称就是参数的名称，返回值类型就是参数的类型。

### 注解参数支持的数据类型
1. 8种基本数据类型
1. String类型
1. Class类型
1. enum类型
1. Annotation类型




