# Netty 介绍

标签（空格分隔）： java-netty

---

[toc]

## 介绍
> Netty是基于Java NIO client-server的网络应用框架，使用Netty可以快速开发网络应用，例如服务器和客户端协议。

### NIO的通信步骤
1. 创建ServerSocketChannel，为其配置非阻塞模式。
1. 绑定监听，配置TCP参数，录入backlog大小等。
1. 创建一个独立的IO线程，用于轮询多路复用器Selector。
1. 创建Selector，将之前创建的ServerSocketChannel注册到Selector上，并设置监听标识位SelectionKey.OP_ACCEPT。
1. 启动IO线程，在循环体中执行Selector.select()方法，轮询就绪的通道。
1. 当轮询到处于就绪状态的通道时，需要进行操作位判断，如果是ACCEPT状态，说明是新的客户端接入，则调用accept方法接收新的客户端。
1. 设置新接入客户端的一些参数，如非阻塞，并将其继续注册到Selector上，设置监听标识位等。
1. 如果轮询的通道标识位是READ，则进行读取，构造Buffer对象等。

### Netty通信步骤
1. 创建两个NIO线程组，一个专门用于网络事件处理（接受客户端的连接），另一个则进行网络通信的读写。
1. 创建一个ServerBootstrap对象，配置Netty的一系列参数，例如接受传出数据的缓存大小等。
1. 创建一个用于实际处理数据的类ChannelInitializer，进行初始化的准备工作，比如设置接受传出数据的字符集、格式以及实际处理数据的接口。
1. 绑定端口，执行同步阻塞方法等待服务器端启动即可。

### [服务端测试代码](https://github.com/scyking/subject/blob/master/src/main/java/server/TestNettyServer.java)

### [客户端测试代码](https://github.com/scyking/subject/blob/master/src/main/java/client/TestNettyClient.java)

## TCP粘包、拆包问题
> TCP是一个“流”协议，所谓流就是没有界限的遗传数据。大家可以想象一下，如果河水就好比数据，他们是连成一片的，没有分界线，TCP底层并不了解上层业务数据的具体含义，它会根据TCP缓冲区的具体情况进行包的划分，也就是说，在业务上一个完整的包可能会被TCP分成多个包进行发送，也可能把多个小包封装成一个大的数据包发送出去，这就是所谓的粘包/拆包问题。

### 解决方案
1. 消息定长，例如每个报文的大小固定为200个字节，如果不够，空位补空格。
1. 在包尾部增加特殊字符进行分割，例如加回车等。
1. 将消息分为消息头和消息体，在消息头中包含表示消息总长度的字段，然后进行业务逻辑的处理。

### Netty 中解决方法
1. 分隔符类：`DelimiterBasedFrameDecoder`（自定义分隔符）
1. 定长：`FixedLengthFrameDecoder`

## Netty编解码技术

1. JBoss的`Marshalling`包
1. google的`Protobuf`
1. 基于`Protobuf`的`Kyro`
1. `MessagePack`框架
