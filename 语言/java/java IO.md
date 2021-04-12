# java IO

标签（空格分隔）： java

---

[toc]

## 基本概念
> 将数据的输入输出抽象为流，流是一组有顺序的，单向的，有起点和终点的数据集合，就像水流。按照流中的最小数据单元又分为字节流和字符流。

### 字节流(InputStream/OutputStream)
> 以 **1 byte** 作为一个数据单元，数据流中最小的数据单元是字节。

### 字符流(Reader/Writer)
> 以 **1 char (2 byte)** 作为一个数据单元，数据流中最小的数据单元是字符。
> Java 中的字符是 Unicode 编码，一个字符占用两个字节。

## NIO

### 区别
- 面向**流**的I/O系统一次一个字节处理数据。一个输入流产生一个字节数据，一个输出流消费一个字节数据。
- NIO 使用**块**I/O的处理方式。每一个操作都在一步中产生或者消费一个数据块。

|IO  |              NIO|
|---
|面向流  |          面向缓冲|
|阻塞IO  |          非阻塞IO|
|无      |          选择器|

### 通道

> Channel是对原I/O包中的流模拟。到任何目的地或来自任何地方的所有数据都必须通过一个Channel对象。

通道类型：

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

### 缓冲区

> Buffer实质是一个容器对象。所有数据的处理都在缓冲区中。发送给一个通道的所有对象都必须先发到缓冲区中。同样，从通道中读取的任何数据都要读到缓冲区中。

缓冲区类型：

- ByteBuffer
- CharBuffer
- IntBuffer
- FloatBuffer 等等。

### Selector 选择器（多路复用）
> 是Java NIO核心组件中的一个，用于检查一个或多个NIO Channel（通道）的状态是否处于可读、可写。
> 如此可以实现单线程管理多个channels,也就是可以管理多个网络链接。
> 使用Selector的好处在于： 使用更少的线程来就可以来处理通道了， 相比使用多个线程，避免了线程上下文切换带来的开销。

1. Selector的使用方法:

- Selector的创建 `java Selector selector = Selector.open();`
- 注册Channel到Selector(Channel必须是非阻塞的) 

    ```
    java channel.configureBlocking(false); 
    SelectionKey key = channel.register(selector, Selectionkey.OP_READ);
    ```
一个SelectionKey键表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系。 

### 读取
1. 从FileInputStream获取Channel。
2. 创建Buffer。
3. 将数据从Channel中读到Buffer中。

### 写入        
1. 从FileOutputStream中获取一个通道。
2. 创建缓冲区，并在其中放入数据。
3. 写入缓冲区中。

## AIO （NIO 2.0）
> 引入了新的异步通道的概念，并提供了异步文件通道和异步套接字通道的实现。

异步通道获取获取操作结果方式：
1. 使用java.util.concurrent.Future类表示异步操作的结果；
2. 在执行异步操作的时候传入一个java.nio.channels。操作完成后胡回调CompletionHandler接口的实现类。

PS: NIO 2.0的异步套接字通道是真正的异步非阻塞I/O，对应于UNIX网络编程中的事件驱动I/O（AIO）。

## IO 与 NIO 示例

```
String PATH = ".\\src\\javanio\\Test.txt";
File file = new File(PATH);
FileInputStream in = new FileInputStream(file);
```
```
// use io
byte[] b = new byte[1024];
in.read(b);
in.close();
```
```
// use nio
FileChannel channel = in.getChannel();
ByteBuffer buffer = ByteBuffer.allocate(1024);
channel.read(buffer);
byte[] b = buffer.array();
in.close();
```



