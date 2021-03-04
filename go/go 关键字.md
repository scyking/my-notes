# go 关键字

标签（空格分隔）： go

---

[toc]

## 关键字或保留字

|||||||
|---
|break|	default|	func|	interface|	select|
|case|	defer|	go|	map|	struct|
|chan|	else|	goto|	package|	switch|
|const|	fallthrough|	if|	range|	type|
|continue|	for|	import|	return|	var|

### defer

> 延迟调用

- defer特性：

1. 关键字 defer 用于注册延迟调用。
2. 这些调用直到 return 前才被执。因此，可以用来做资源清理。
3. 多个defer语句，按**先进后出**的方式执行。
4. defer语句中的变量，在defer声明时就决定了。

- defer用途：

1. 关闭文件句柄
2. 锁资源释放
3. 数据库连接释放

### fallthrough

> 用在switch中，且只能在每个case分支中最后一行出现。
> 如果这个case分支被执行，将会继续执行下一个case分支，而不会去判断下一个分支的case条件是否成立。

## 预定义标识符

> 预定义标识符全部列出在源码的builtin包中，只有一个builtin.go文件，并且都有注释说明

### 内建常量

```
true false iota nil
```
### 内建类型: 

```
int int8 int16 int32 int64
uint uint8 uint16 uint32 uint64 uintptr
float32 float64 complex128 complex64
bool byte rune string error
```

- `byte`：`uint8`别名
- `rune`：`int32`别名
- `uintptr`：无符号整数（没有指定大小，足够容纳指针）
- `complex64`：复数（32位实数和虚数）
- `complex128`：复数（64位实数和虚数）
### 内建函数

```
make len cap new append copy close delete
complex real imag
panic recover
print println
```

- `close`： 用于发送方关闭chan，仅适用于双向或发送通道
- `len`、`cap`： 用于获取数组、Slice、map、string、chan类型数据的长度或数量，`len`返回长度、`cap`返回容量
- `new`：用于值类型、用户定义类型的内存分配，new(T)将分配T类型零值返回其指向T类型的指针
- `make`：用于引用类型（Slice、map、chan）内存分配返回初始化的值，不同类型使用有所区别

    ```
    make(chan int)          //创建无缓冲区的通道  
    make(chan int,10)       //创建缓冲区为10的通道  
    make([]int,1,5)         //创建长度为1，容量为5的slice  
    make([]int,5)           //创建容量长度都为5的slice  
    make(map[int] int)      //创建不指定容量的map
    make(map[int] int,2)    //创建容量为2的map  
    ```
- `copy`、`append`：用于复制slice与为slice追加元素
- `print`、`println`：用于打印输出
- `panic`、`recover`：用于错误处理
- `delete`：用于删除map中的指定key
- `complex`、`real`、`imag`：复数操作

















