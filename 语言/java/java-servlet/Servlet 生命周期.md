# Servlet 生命周期

标签（空格分隔）： java-servlet

---

[toc]

## 生命周期

> Servlet 加载 —> 实例化 —> 服务 —> 销毁。

- `init（）`
> 在Servlet的生命周期中，仅执行一次init()方法。它是在服务器装入Servlet时执行的，负责初始化Servlet对象。可以配置服务器，以在启动服务器或客户机首次访问Servlet时装入Servlet。无论有多少客户机访问Servlet，都不会重复执行init（）。

- `service（）`
> 它是Servlet的核心，负责响应客户的请求。每当一个客户请求一个HttpServlet对象，该对象的Service()方法就要调用，而且传递给这个方法一个“请求”（ServletRequest）对象和一个“响应”（ServletResponse）对象作为参数。在HttpServlet中已存在Service()方法。默认的服务功能是调用与HTTP请求的方法相应的do功能。

- `destroy（）`
> 仅执行一次，在服务器端停止且卸载Servlet时执行该方法。当Servlet对象退出生命周期时，负责释放占用的资源。一个Servlet在运行service()方法时可能会产生其他的线程，因此需要确认在调用destroy()方法时，这些线程已经终止或完成。

## Tomcat 与 Servlet

```sequence
participant Web Client
participant Servlet Container
participant HttpServletRequest
participant HttpServletResponse
participant HttpServlet

Web Client->>Servlet Container: 1. http request
Servlet Container->>Servlet Container: 2. incept http request
Servlet Container->>HttpServletRequest: 3. new instance
Servlet Container->>HttpServletResponse: 4. new instance
Servlet Container->>HttpServlet: 5. invoke service method
HttpServlet->>HttpServletRequest: 6. get request information
HttpServlet->>HttpServletResponse: 7. set response information
Servlet Container->>Web Client: 8. http response
```

### Tomcat 顶层架构
> 最顶层是`Server`，代表整个服务器。`Server` 至少包含一个 `Service`。
>> `Service` 包含`Connector`和`Container`两部分。一个`Service`只有一个`Container`，但是可以有多个`Connector`，这是因为一个服务可以有多个连接。

- `Connector` 用于处理连接相关的事情，并提供`Socket`与`Request`和`Response`相关的转化；
- `Container` 用于封装和管理`Servlet`，以及具体处理`Request`请求。

### Connector 架构
1. `Connector`如何接受请求的？
    > `Connector`使用`ProtocolHandler`来处理请求的，不同的`ProtocolHandler`代表不同的连接类型。比如：`Http11Protocol`使用的是普通`Socket`来连接的，`Http11NioProtocol`使用的是`NioSocket`来连接的。
1. 如何将请求封装成`Request`和`Response`，并交给`Container`进行处理的？
    > `ProtocolHandler`由包含了三个部件：`Endpoint`、`Processor`、`Adapter`。
    - `Endpoint` 实现了`TCP/IP`协议，用来处理底层`Socket`的网络连接
    - `Processor`实现了`HTTP`协议，用于将`Endpoint`接收到的`Socket`封装成`Request`
    - `Adapter`用于将`Request`交给`Container`进行具体的处理
    > `Endpoint`的抽象实现`AbstractEndpoint`里面定义的`Acceptor`和`AsyncTimeout`两个内部类和`一个Handler`接口。
    - `Acceptor`用于监听请求
    - `AsyncTimeout`用于检查异步`Request`的超时
    - `Handler`用于处理接收到的`Socket`，在内部调用`Processor`进行处理。

### Container 架构
1. `Container`组成
    - `Engine`：引擎，用来管理多个站点，一个`Service`最多只能有一个`Engine`；
    - `Host`：代表一个站点，也可以叫虚拟主机，通过配置`Host`就可以添加站点；
    - `Context`：代表一个应用程序，对应着平时开发的一套程序，或者一个`WEB-INF`目录以及下面的`web.xml`文件；
    - `Wrapper`：每一`Wrapper`封装着一个`Servlet`。

1. `Container` 如何处理请求
    > `Container`处理请求是使用`Pipeline-Valve`管道来处理的。
    > `Pipeline-Valve`是责任链模式。责任链模式是指在一个请求处理的过程中有很多处理者依次对请求进行处理，每个处理者负责做自己相应的处理，处理完之后将处理后的请求返回，再让下一个处理着继续处理。
    > 与普通责任链区别：
    - 每个`Pipeline`都有特定的`Valve`，而且是在管道的最后一个执行，这个`Valve`叫做`BaseValve`，`BaseValve`是不可删除的
    - 在上层容器的管道的`BaseValve`中会调用下层容器的管道
    > 处理流程：
    - `Container`在接收到请求后会首先调用最顶层容器的`Pipeline`（即`EnginePipeline`）来处理
    - 在`Engine`的管道中依次会执行`EngineValve1`、`EngineValve2`等等，最后会执行`StandardEngineValve`，在`StandardEngineValve`中会调用`Host`管道，然后再依次执行`Host`的`HostValve1`、`HostValve2`等，最后在执行`StandardHostValve`，然后再依次调用`Context`的管道和`Wrapper`的管道，最后执行到`StandardWrapperValve`。
    - 当执行到`StandardWrapperValve`的时候，会在`StandardWrapperValve`中创建`FilterChain`，并调用其`doFilter`方法来处理请求，这个`FilterChain`包含着我们配置的与请求相匹配的`Filter`和`Servlet`，其`doFilter`方法会依次调用所有的`Filter`的`doFilter`方法和`Servlet`的`service`方法，这样请求就得到了处理。
    - 当所有的`Pipeline-Valve`都执行完之后，并且处理完了具体的请求，这个时候就可以将返回的结果交给`Connector`了，`Connector`在通过`Socket`的方式将结果返回给客户端。

### [示例代码](https://github.com/scyking/subject/tree/master/src/main/java/server/TestTomcat.java)

## Servlet工作原理
- Servlet接收和响应客户请求的过程。首先客户发送一个请求，Servlet是调用service()方法对请求进行响应的，通过源代码可见，service()方法中对请求的方式进行了匹配，选择调用doGet,doPost等这些方法，然后再进入对应的方法中调用逻辑层的方法，实现对客户的响应。在Servlet接口和GenericServlet中是没有doGet（）、doPost（）等等这些方法的，HttpServlet中定义了这些方法，但是都是返回error信息，所以，我们每次定义一个Servlet的时候，都必须实现doGet或doPost等这些方法。
- 每一个自定义的Servlet都必须实现Servlet的接口，Servlet接口中定义了五个方法，其中比较重要的三个方法涉及到Servlet的生命周期，分别是上文提到的init(),service(),destroy()方法。GenericServlet是一个通用的，不特定于任何协议的Servlet,它实现了Servlet接口。而HttpServlet继承于GenericServlet，因此HttpServlet也实现了Servlet接口。所以我们定义Servlet的时候只需要继承HttpServlet即可。
- Servlet接口和GenericServlet是不特定于任何协议的，而HttpServlet是特定于HTTP协议的类，所以HttpServlet中实现了service()方法，并将请求ServletRequest、ServletResponse
强转为HttpRequest 和 HttpResponse。

### 创建Servlet对象的时机

1. Servlet容器启动时
    > 读取web.xml配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，同时将ServletConfig对象作为参数来调用Servlet对象的init方法。
1. 在Servlet容器启动后
    > 客户首次向Servlet发出请求，Servlet容器会判断内存中是否存在指定的Servlet对象，如果没有则创建它，然后根据客户的请求创建HttpRequest、HttpResponse对象，从而调用Servlet对象的service方法。
1. Servlet
    > 同一个类型的Servlet对象在Servlet容器中以单例的形式存在。Servlet 容器处理由多个线程产生的多个请求，每个线程执行一个单一的 Servlet 实例的 service() 方法。
