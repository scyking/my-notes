# Tomcat 系统架构

标签（空格分隔）： 计算机网络

---

[toc]

## Server

> Tomcat 最顶层容器，代表整个服务器。
> 一个`Server`至少包含一个`Service`。

## Service

> 包含两部分，一个`Container`和多个`Connector`，用于提供具体服务。

## Connector

> 用于处理连接相关的事情，并提供`Socket`与`Request`和`Response`相关的转化；

### 架构组成

> `Connector`使用`ProtocolHandler`处理请求，不同的`ProtocolHandler`代表不同的连接类型。

- `Endpoint`：用来处理底层`Socket`的网络连接。
    1. `Acceptor`，用于监听请求
    2. `AsyncTimeout`，用于检查异步`Request`的超时
    3. `Handler`，用于处理接收到的`Socket`，在内部调用`Processor`进行处理
- `Processor`：用于将`Endpoint`接收到的`Socket`封装成`Request`。
- `Adapter`：用于将`Request`交给`Container`进行具体的处理。

## Container

> 用于封装和管理`Servlet`，以及具体处理`Request`请求。

### 四个子容器

- `Engine`

> 引擎，用来管理多个站点，一个`Container`最多只能有一个`Engine`。

- `Host`

> 站点，也可以叫虚拟主机，通过配置`Host`就可以添加站点。

- `Context`

> 代表一个应用程序，对应着平时开发的一套程序，或者一个`WEB-INF`目录以及下面的`web.xml`文件。

- `Wrapper`

> 每一Wrapper封装着一个Servlet。

### 请求处理

> 使用`Pipeline-Valve`管道，采用责任链模式处理请求。

1. `Container`在接收到请求后，会调用最顶层容器的`Engine`的管道（`EnginePipeline`）来处理。

1. 在`Engine`的管道中依次执行`EngineValve1`、`EngineValve2`等。最后会执行`StandardEngineValve`，调用`Host`管道。

1. 在`Host`管道中依次执行`HostValve1`、`HostValve2`等。最后会执行`StandardHostValve`，依次调用`Context`管道、`Wrapper`管道，最后执行到`StandardWrapperValve`。

1. 当执行到`StandardWrapperValve`的时候，会在`StandardWrapperValve`中创建`FilterChain`，并调用其`doFilter`方法来处理请求。`FilterChain`包含配置与请求相匹配的`Filter`和`Servlet`，其`doFilter`方法会依次调用所有的`Filter`的`doFilter`方法和`Servlet`的`service`方法来处理请求。

1. 当所有的`Pipeline-Valve`都执行结束，并且处理完具体的请求后，会将返回的结果交给`Connector`，`Connector`通过`Socket`的方式将结果返回给客户端。

## 工作原理

### 配置文件

```
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">
   <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
   <Listener className="org.apache.catalina.core.JasperListener" />
   <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
   <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
   <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
   <GlobalNamingResources>
     <Resource name="UserDatabase" auth="Container"
               type="org.apache.catalina.UserDatabase"
               description="User database that can be updated and saved"
               factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
               pathname="conf/tomcat-users.xml" />
   </GlobalNamingResources>
   <Service name="Catalina">
     <Connector port="8080" protocol="HTTP/1.1"
                connectionTimeout="20000"
                redirectPort="8443" />
     <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
     <Engine name="Catalina" defaultHost="localhost">
       <Realm className="org.apache.catalina.realm.LockOutRealm">
         <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
                resourceName="UserDatabase"/>
       </Realm>
       <Host name="localhost"  appBase="webapps"
             unpackWARs="true" autoDeploy="true">
         <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                prefix="localhost_access_log." suffix=".txt"
                pattern="%h %l %u %t &quot;%r&quot; %s %b" />
       </Host>
     </Engine>
   </Service>
</Server>
```

### 整体流程

1. 用户在客户端点击网页时，将请求发送到本机端口`8080`，会被在那里监听的`Coyote HTTP/1.1 Connector`获得。

1. `Connector`连接器把该请求交给它所在的`Service`的`Engine`来处理，并等待`Engine`的回应的结果。

1. `Engine`获得请求的资源，如`index.jsp`，匹配所有的虚拟主机`Host`。

1. `Engine`匹配到名为`localhost`的`Host`。Host获得请求资源的信息，如`/test/index.jsp`，匹配它所拥有的所有`Context`。

1. `path="/test"`的`Context`获得请求`/index.jsp`，在它的`mapping table`中寻找出对应的`Servlet`。

1. 构造`HttpServletRequest`对象和`HttpServletResponse`对象，作为参数调用`JspServlet`的`doGet()`或`doPost()`。执行业务逻辑、数据存储等程序。

1. `Context`把执行完之后的`HttpServletResponse`对象返回给`Host`。

1. `Host`把`HttpServletResponse`对象返回给`Engine`。

1. `Engine`把`HttpServletResponse`对象返回`Connector`。

1. `Connector`把`HttpServletResponse`对象返回给客户端。
