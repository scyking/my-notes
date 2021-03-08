# java servlet 生命周期

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

## Servlet工作原理
- Servlet接收和响应客户请求的过程。首先客户发送一个请求，Servlet是调用service()方法对请求进行响应的，通过源代码可见，service()方法中对请求的方式进行了匹配，选择调用doGet,doPost等这些方法，然后再进入对应的方法中调用逻辑层的方法，实现对客户的响应。在Servlet接口和GenericServlet中是没有doGet（）、doPost（）等等这些方法的，HttpServlet中定义了这些方法，但是都是返回error信息，所以，我们每次定义一个Servlet的时候，都必须实现doGet或doPost等这些方法。

- 每一个自定义的Servlet都必须实现Servlet的接口，Servlet接口中定义了五个方法，其中比较重要的三个方法涉及到Servlet的生命周期，分别是上文提到的init(),service(),destroy()方法。GenericServlet是一个通用的，不特定于任何协议的Servlet,它实现了Servlet接口。而HttpServlet继承于GenericServlet，因此HttpServlet也实现了Servlet接口。所以我们定义Servlet的时候只需要继承HttpServlet即可。

- Servlet接口和GenericServlet是不特定于任何协议的，而HttpServlet是特定于HTTP协议的类，所以HttpServlet中实现了service()方法，并将请求ServletRequest、ServletResponse
强转为HttpRequest 和 HttpResponse。

### 创建Servlet对象的时机

1. Servlet容器启动时
读取web.xml配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，同时将ServletConfig对象作为参数来调用Servlet对象的init方法。
1. 在Servlet容器启动后
客户首次向Servlet发出请求，Servlet容器会判断内存中是否存在指定的Servlet对象，如果没有则创建它，然后根据客户的请求创建HttpRequest、HttpResponse对象，从而调用Servlet对象的service方法。
1. Servlet
同一个类型的Servlet对象在Servlet容器中以单例的形式存在。Servlet 容器处理由多个线程产生的多个请求，每个线程执行一个单一的 Servlet 实例的 service() 方法。
