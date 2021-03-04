# Spring Cloud Netflix

标签： spring-cloud

--- 

> [Spring Cloud Netflix 官网介绍](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.1.0.RC3/single/spring-cloud-netflix.html#netflix-eureka-client-starter)
> Spring Cloud Netflix 核心模块包括：Eureka（服务发现）、Zuul（智能路由）、Hystrix（断路器）、Ribbon（客户端负载均衡

[toc]

## 服务发现：Eureka

### 构建 Eureka Server（服务注册中心）

Eureka Server 本身相当于一个微服务。

- 添加依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```    
PS：使用IDEA创建Spring Boot项目时，勾选`Cloud Discovery`中`Eureka Server`项即可。

- 添加配置

```
server:
  port: 8761 # 服务监听端口
# 切记不可添加其它路径，即使一样（即“/eureka”）也不行（PS：一样的就是神坑，看都看不出来）  
#  servlet:
#    context-path: /eureka

spring:
  security:
    user:
      name: admin
      password: 123456

eureka:
  instance:
    hostname: 127.0.0.1 #服务注册中心IP地址
  client:
    registerWithEureka: false # 不使用 eureka 服务进行注册
    fetchRegistry: false # 不在本地缓存注册表信息
    serviceUrl: #服务注册中心的配置内容，指定服务注册中心的位置
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
# 本地测试，为提高启动速度注释掉
# server:
#   waitTimeInMsWhenSynEmpty: 5 # 在服务器接收请求之前等待的初始时间
```
PS：`sping boot 2.x` ，配置规范有更改，自行查看官网查看。
            
- 修改启动类
添加注解 `@@EnableEurekaServer`，如下：
```
@SpringBootApplication
@EnableEurekaServer       
public class EurekaServerApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### Eureka Client：注册服务（生产者，服务提供者）

服务提供者，即各个微服务。

- 添加依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

PS：使用IDEA创建Spring Boot项目时，勾选`Cloud Discovery`中`Eureka Server`项、`Web`中`Web`项即可。

- 添加配置

```
server:
  port: 8081  #服务端口号

spring:
  application:
    name: service-provider # 服务名称--调用的时候根据名称来调用该服务的方法
  security:
    user:
      name: admin
      password: 123456

eureka:
  instance:
    prefer-ip-address: true
    hostname: 127.0.0.1 # 服务注册中心IP地址
  client:
    service-url:
      defaultZone: http://${spring.security.user.name}:${spring.security.user.password}@${eureka.instance.hostname}:8761/eureka/ # 注册中心地址

```

- 修改启动类
添加注解 `@EnableEurekaClient`，如下：
```
@SpringBootApplication
@EnableEurekaClient          
public class EurekaClient1Application {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClient1Application.class, args);
    }
}
```
PS：注册中心是`Eureka`使用`@EnableEurekaClient`，其他注册中心使用`@EnableDiscoveryClient`。
- `@EnableDiscoveryClient` 基于 `spring-cloud-commons`;
- `@EnableEurekaClient` 基于 `spring-cloud-netflix`;

- 添加服务
```
@RestController
@RequestMapping
public class TestController {

    @GetMapping("/test")
    public String test() {
        return "Hello Client 1!";
    }
}
```

### Eureka Client：发现服务（消费者，服务调用者）

- 添加同注册服务的依赖、配置、启动类
- 添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
- 修改启动类

添加`@EnableFeignClients`，启用 `feign` 进行远程调用。

- `feign` 调用实现
```
/**
 * service-provider 服务远程调用
 */
@FeignClient(name = "service-provider") // 获取服务名称
public interface Client1Remote {

    @GetMapping("/test")  // 对应的服务接口
    String test();
}
```

- 调用远程服务
```
@RestController
@RequestMapping
public class TestController {

    @Autowired
    Client1Remote client1Remote;

    @GetMapping("/")
    public Object test() {
        Object obj = client1Remote.test();
        return obj.toString();
    }
}
```
PS:注意结果返回类型，不一致会导致无法解析而报错。

### 缺点

- 适用场景有局限，服务提供者对我接口不能发生变化，会直接影响到消费者。

### 问题解决

1. 客户端注册服务是报错“Cannot execute request on any known server”。

- 方法一

    在服务端添加如下配置（spring boot 2.X 会出现该问题）：
    ```
    @Configuration
    @EnableWebSecurity
    public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

        /**
        * Securing The Eureka Server
        *
        * @throws Exception
        */
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.csrf().ignoringAntMatchers("/eureka/**");
            super.configure(http);
        }
    }
    ```
- 方法二
    - 检查注册中心服务地址是否配置正确；
    - 检查注册中心服务是否启动；
    - 神坑，检查服务注册中心`context-path`是否添加额外配置项，导致与配置中心地址不匹配；

## 熔断器：Hystrix

`Hystrix` 是 `Netflix` 开源的一个延迟和容错库，用于隔离访问远程系统、服务或第三方库，防止级联失败，从而提高系统的可用性与容错性。

### Hystrix

- 添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
- 修改启动类
添加`@EnableHystrix`注解，如下：
```
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
@EnableHystrix
public class EurekaClient2Application {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClient2Application.class, args);
    }
}
```

- 修改Controller
添加测试接口，如下：
```
@GetMapping("/obj")
@HystrixCommand(fallbackMethod = "getObjectStrFallback")
public Object getObjectStr() {
    return 1 / 0;
}

public String getObjectStrFallback() {
    return "getObjectStr failed !";
}
```
访问`http://127.0.0.1:8082/obj`，返回“getObjectStr failed !”。

### Feign Hystrix

熔断只是作用在服务调用这一端，因此修改服务调用服务即可。

- 修改配置文件
```
feign:
  hystrix:
    enabled: true
```

- 创建回调类
```
@Component
public class Client1RemoteHystrix implements Client1Remote {
    @Override
    public String test() {
        return "Client1Remote, this message send failed !";
    }
}
```

- 添加`fallback`属性
```
/**
 * service-provider 服务远程调用
 */
@FeignClient(name = "service-provider", fallback = Client1RemoteHystrix.class)
public interface Client1Remote {

    @GetMapping("/test")
    String test();
}
```
- 测试
访问`http://127.0.0.1:8082/`，返回“Hello Client 1!”。
停掉服务提供者服务，再次访问，返回“Client1Remote, this message send failed !”。
说明熔断成功。

### 熔断监控：Hystrix Dashboard

Hystrix-dashboard 是一款针对 Hystrix 进行实时监控的工具，通过 Hystrix Dashboard 我们可以在直观地看到各 Hystrix Command 的请求响应时间, 请求成功率等数据。

- 添加依赖
```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
<!--actuator用于应用监控管理-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

- 修改配置
```
management:
  endpoint:
    health:
      show-details: always
```
PS:不添加会报`Unable to connect to Command Metric Stream.`错误。

- 修改启动类
添加`@EnableHystrixDashboard`注解，如下：
```
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
@EnableHystrixDashboard
public class EurekaClient2Application {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClient2Application.class, args);
    }
}
```

- 测试
访问`http://127.0.0.1:8082/hystrix`，进入Hystrix Dashboard页面。

### 熔断聚合监控：Turbine

Turbine 把多个 hystrix.stream 的内容聚合为一个数据源，供 Dashboard 展示。

- 添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

- 添加配置
```
spring:
  application:
    name: hystrix-dashboard-turbine
  security:
    user:
      name: admin
      password: 123456

server:
  port: 8003

turbine:
  appConfig: service-consumer # 配置Eureka中的serviceId列表，表明监控哪些服务
  aggregator:
    clusterConfig: default
  clusterNameExpression: new String("default")

eureka:
  instance:
    hostname: 127.0.0.1
  client:
    serviceUrl:
      defaultZone: http://${spring.security.user.name}:${spring.security.user.password}@${eureka.instance.hostname}:8761/eureka/
```

- 修改启动项
添加`@EnableTurbine`注解，如下：
```
@SpringBootApplication
@EnableHystrixDashboard
@EnableTurbine
public class ServiceTurbineApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceTurbineApplication.class, args);
    }
}
```
- 测试
访问`http://127.0.0.1:8003/turbine.stream`，显示`ping`信息。


### 问题解决

1. 熔断监控，输入地址后，报`Unable to connect to Command Metric Stream.`。
在启动类中，添加如下：
```
@Bean
public ServletRegistrationBean getServlet(){
    HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
    registrationBean.setLoadOnStartup(1);
    registrationBean.addUrlMappings("/actuator/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    return registrationBean;
}
```

## 智能路由：Zuul

提供动态路由，监控，弹性，安全等的边缘服务。

### 简单搭建

- 添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```
- 修改配置文件
```
spring:
  application:
    name: gateway-service-zuul

server:
  port: 8888

# 这里的配置表示，访问 /it/** 直接重定向到 http://www.baidu.com/**
zuul:
  routes:
    baidu:
      path: /it/**
      url: http://www.baidu.com/
```

- 修改启动类
添加`@EnableZuulProxy`注解，如下：
```
@SpringBootApplication
@EnableZuulProxy
public class ApiGatewayZuulApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayZuulApplication.class, args);
    }
}
```

- 测试
访问`http://127.0.0.1:8888/it/`，会自动跳转到`http://www.baidu.com/`。

### 服务化

通过url映射的方式来实现zull的转发有局限性，比如每增加一个服务就需要配置一条内容，另外后端的服务如果是动态来提供，就不能采用这种方案来配置了。

- 添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

- 修改配置
```
zuul:
  routes:
    baidu:
      path: /it/**
      url: http://www.baidu.com/
    api-a:
      path: /producer/**
      serviceId: service-provider # 注册中心注册的服务名称
```

- 测试
访问`http://127.0.0.1:8888/producer/test`，返回结果“Hello Client 1!”。

- 默认路由规则
`http://ZUUL_HOST:ZUUL_PORT/`，微服务在Eureka上的`serviceId/**`会被转发到serviceId对应的微服务。
注释掉路由配置，访问`http://127.0.0.1:8888/service-provider/test`，得到相同结果。

## 客户端负载均衡：Ribbon

### 声明式 rest 调用：Feign

- 启用 `feign` 进行远程调用，引入方式同 “Eureka 服务发现”。

- 默认负载方式是轮询。

## 分布式链路监控：Spring Cloud Sleuth

### 链路监控 -- 服务端

使用 Spring Boot 2.x 版本后，官方不推荐自行定制编译（提供编译好的jar）。

- 所以官方提供了一键脚本
```
curl -sSL https://zipkin.io/quickstart.sh | bash -s
java -jar zipkin.jar
```
- 如果用 Docker 的话，直接
```
docker run -d -p 9411:9411 openzipkin/zipkin
```
- 测试
启动后访问`http://127.0.0.1:9411`，进入监控页面则成功。

### 链路监控 -- 客户端

- 添加依赖
```
<!-- sleuth配置 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<!-- 引入zipkin  -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```
- 修改配置文件
添加如下内容：
```
spring:
  sleuth:
    web:
      client:
        enabled: true
    sampler:
        probability: 1.0 # 将采样比例设置为 1.0，也就是全部都需要。默认是 0.1
  zipkin:
    base-url: http://127.0.0.1:9411/  # 指定了 Zipkin 服务器的地址
```

## Spring Cloud Config

### Server 端

- 添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
- 修改配置文件
添加如下：
```
spring:
  application:
    name: config-server # 注册服务名
  cloud:
    config:
      server:
        git:
          uri:    # 配置git仓库的地址
          search-paths: /dev # git仓库地址下的相对地址，可以配置多个，用,分割。
          username:  # git仓库的账号
          password: # git仓库的密码
          default-label: master 
```
- 修改启动类
添加注解`@EnableConfigServer`，如下：
```
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```
- 测试
启动成功后，仓库中的配置文件会被转换成web接口，访问可以参照以下的规则：
```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```
在git上`/dev`目录下创建`application-dev.yml`文件。访问`http://127.0.0.1:9999/application-dev.yml`，获取到配置文件内容，说明成功。

### Client 端

- 添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```
- 修改配置文件
修改`bootstrap.yml`，如下：
```
# 与 spring-cloud 相关的属性必须配置在bootstrap.yml，config 部分内容才能被正确加载。
# 因为 config 的相关配置会先于application.yml，而bootstrap.yml 的加载也是先于application.yml。

spring:
  cloud:
    config:
      name: application # 对应{application}部分
      profile: dev # 对应{profile}部分
      # uri: http://127.0.0.1:9999/ # 配置中心的具体地址
      label: master # 对应git的分支。如果配置中心使用的是本地存储，则该参数无用
      discovery:
        enabled: true
        service-id: config-server # 指定配置中心的service-id，便于扩展为高可用配置集群。
```

PS: 通过`service-id`指定配置中心，注册中心配置需提取到`bootstrap.yml`中。

## Spring Cloud Stream

> 数据流操作开发包，可用于解耦消息中间件。

Spring Cloud Stream 绑定消息通道的默认实现

- Sink：通过指定消费消息的目标来标识消息使用者的约定。
- Source：与Sink相反，用于标识消息生产者的约定。
- Processor：集成了Sink和Source的作用，标识消息生产者和使用者

源码分别如下:

```
public interface Sink {
    String INPUT = "input";

    @Input("input")
    SubscribableChannel input();
}

public interface Source {
    String OUTPUT = "output";

    @Output("output")
    MessageChannel output();
}

public interface Processor extends Source, Sink {
}
```
PS：拥有多条管道的时候不能有输入输出管道名相同的，否则会出现发送消息被自己接收或报错的情况。

### RabbitMQ 

- 添加依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

- 生产者



- 消费者


## 问题汇总

1. `Method has too many Body parameters`，使用feign报错。
    接口参数需添加相应Http注解（不可省略）。注意Feign中可以有多个@RequestParam，但只能有不超过一个@RequestBody。







    

    





