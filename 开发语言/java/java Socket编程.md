# java Socket编程

标签（空格分隔）： java

---

[toc]

## 网络协议

- `TCP`（Transmission Control Protocol，传输控制协议）
    > 一种面向连接的、可靠的、基于字节流的传输层通信协议，`TCP` 层是位于 `IP` 层之上，应用层之下的中间层。`TCP` 保障了两个应用程序之间的可靠通信。通常用于互联网协议，被称 `TCP / IP`。

- `UDP` （User Datagram Protocol，用户数据报协议）
    > 一个提供了应用程序之间要发送数据的数据报，位于`OSI`模型传输层的无连接协议。由于`UDP`缺乏可靠性且属于无连接协议，所以应用程序通常必须容许一些丢失、错误或重复的数据包。

## 套接字（Socket）
> 套接字使用`TCP`提供了两台计算机之间的通信机制。
> 客户端程序创建一个套接字，并尝试连接服务器的套接字。
> 当连接建立时，服务器会创建一个 `Socket` 对象。客户端和服务器现在可以通过对 `Socket` 对象的写入和读取来进行通信。

### 通信过程
> 以`TCP Socket`通信过程举例，就是客户端与服务器进行TCP数据交互。

1. 系统分配资源，服务端开启Socket进程，对特定端口号进行监听
1. 客户端针对服务端IP进行特定端口的连接
1. 连接建立，开始通信
1. 通信完成，关闭连接

<img src="https://github.com/scyking/my-pics/blob/master/md/java-socket/java-socket-1.png" width="50%">

### java 调用底层Linux Socket接口实现
> 创建一个TCP连接时，需要首先实例化`java`的`ServerSocket`类，其中封装了底层的`socket（）`方法、`bind()`方法、`listen（）`方法。

- `socket()`方法是JVM对Linux API的调用
1. 创建`socket`结构体
1. 创建`tcp_sock`结构体,刚创建完的`tcp_sock`的状态为:`TCP_CLOSE`
1. 创建文件描述符与`socket`绑定

- `bind ()` ：可以理解为，绑定到系统能够找到的地方
1. 将当前网络命名空间名和端口存到`bhash()`。

- `listen()`：验证连接请求的端口是否被开启
1. 检查侦听端口是否存在bhash中
2. 初始化csk_accept_queue
3. 将tcp_sock指针存放到listening_hash表

- `accpet()`　　
1. 调用accept方法
2. 创建socket(创建新的准备用于连接客户端的socket)
3. 创建文件描述符
4. 阻塞式等待(csk_accept_queue)获取sock

    > 在listen阶段,会为侦听的sock初始化csk_accept_queue,此时这个queue为空，所以accept()方法会在此时阻塞住,直到后面有客户端成功握手后,这个queue才有sock.如果csk_accept_queue不为空,则返回一个sock.后续的逻辑如如下:
    1. 取出sock
    1. socket与sock互相关联
    1. socket与文件描述符关联
    1. 将socket返回给线程

<img src="https://github.com/scyking/my-pics/blob/master/md/java-socket/java-socket-2.png" width="50%">

### java 使用套接字建立TCP连接过程
1. 服务器实例化一个 `ServerSocket` 对象，表示通过服务器上的端口通信。
1. 服务器调用 `ServerSocket` 类的 `accept()` 方法，该方法将一直等待，直到客户端连接到服务器上给定的端口。
1. 服务器正在等待时，一个客户端实例化一个 `Socket` 对象，指定服务器名称和端口号来请求连接。
1. `Socket` 类的构造函数试图将客户端连接到指定的服务器和端口号。如果通信被建立，则在客户端创建一个 `Socket`  对象能够与服务器进行通信。
1. 在服务器端，`accept()` 方法返回服务器上一个新的 `socket` 引用，该 `socket` 连接到客户端的 `socket`。

### [服务端示例](https://github.com/scyking/subject/blob/master/src/main/java/server/TestSocketServer.java)

### [客户端示例](https://github.com/scyking/subject/blob/master/src/main/java/client/TestSocketClient.java)

## [I/O介绍](java%20IO.md)

### 同步与异步
> 关注的是消息通信机制 (synchronous communication/ asynchronous communication)

- 同步
就是在发出一个*调用*时，在没有得到结果之前，该*调用*就不返回。但是一旦调用返回，就得到返回值了。换句话说，就是由*调用者*主动等待这个*调用*的结果。
- 异步
与同步相反，*调用*在发出之后，这个调用就直接返回了，所以没有返回结果。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在*调用*发出后，*被调用者*通过状态、通知来通知调用者，或通过回调函数处理这个调用。

### 阻塞与非阻塞
> 关注的是程序在等待调用结果（消息，返回值）时的状态

- 阻塞
指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
- 非阻塞
指在不能立刻得到结果之前，该调用不会阻塞当前线程。

### 类别
1. BIO 编程（Blocking IO）：同步阻塞的编程方式

1. NIO 编程（New IO）：同步非阻塞的编程方式
    > NIO本身是基于事件驱动思想来完成的，其主要想解决的是BIO的大并发问题，NIO基于Reactor，当socket有流可读或可写入socket时，操作系统会相应的通知引用程序进行处理，应用再将流读取到缓冲区或写入操作系统。
    > 在NIO的处理方式中，当一个请求来的话，开启线程进行处理，可能会等待后端应用的资源(JDBC连接等)，其实这个线程就被阻塞了，当并发上来的话，还是会有BIO一样的问题。

1. AIO编程（Asynchronous IO）：异步非阻塞的编程方式
    > 与NIO不同，当进行读写操作时，只须直接调用API的read或write方法即可。