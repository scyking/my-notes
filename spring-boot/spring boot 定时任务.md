# spring boot 定时任务
---

标签：spring-boot

---


## spring boot定时任务实现

- 在启动类上添加`@EnableScheduling`注解
- 创建定时任务，在相应方法上添加`@Scheduled`注解


## ejb3定时任务实现

1. 创建一个sessionbean 和对应的接口 
2. 获取一个sessionContext对象 
3. 在sessionContext中get到TimerService 
4. 在TimeService中获取到Timer对象,创建该对象时候第一个参数代表定时任务开始执行的时间(精确到毫秒),第二个参数代表建个多久,第三个为传入的参数(这个参数在执行任务的时候可以取到) 
5. 写一个方法,名称规则为public xxx(Timer timer) 
6. 用annotation Timeout修饰这个方法 

# spring cloud

简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等。

## 核心

### Spring Cloud Netflix

- Netflix Eureka

	服务中心，云端服务发现，一个基于 REST 的服务，用于定位服务，以实现云端中间层服务发现和故障转移。

- Netflix Hystrix

	熔断器，容错管理工具，旨在通过熔断机制控制服务和第三方库的节点,从而对延迟和故障提供更强大的容错能力。

- Netflix Zuul

	Zuul 是在云平台上提供动态路由,监控,弹性,安全等边缘服务的框架。

- Netflix Archaius

	配置管理API，包含一系列配置管理API，提供动态类型化属性、线程安全配置操作、轮询框架、回调机制等功能。

### Spring Cloud Config

俗称的配置中心，配置管理工具包，让你可以把配置放到远程服务器，集中化管理集群配置，目前支持本地存储、Git以及Subversion。

### Spring Cloud Bus

事件、消息总线，用于在集群（例如，配置变化事件）中传播状态变化，可与Spring Cloud Config联合实现热部署。

### Spring Cloud for Cloud Foundry

Cloud Foundry是VMware推出的业界第一个开源PaaS云平台，它支持多种框架、语言、运行时环境、云平台及应用服务，使开发人员能够在几秒钟内进行应用程序的部署和扩展，无需担心任何基础架构的问题。

### Spring Cloud Cluster

Spring Cloud Cluster将取代Spring Integration。提供在分布式系统中的集群所需要的基础功能支持，如：选举、集群的状态一致性、全局锁、tokens等常见状态模式的抽象和实现。

### Spring Cloud Consul

Consul 是一个支持多数据中心分布式高可用的服务发现和配置共享的服务软件,由 HashiCorp 公司用 Go 语言开发, 基于 Mozilla Public License 2.0 的协议进行开源. Consul 支持健康检查,并允许 HTTP 和 DNS 协议调用 API 存储键值对.

Spring Cloud Consul 封装了Consul操作，consul是一个服务发现与配置工具，与Docker容器可以无缝集成。

## 其它

### Spring Cloud Security

基于spring security的安全工具包，为你的应用程序添加安全控制。

### Spring Cloud Sleuth

日志收集工具包，封装了Dapper和log-based追踪以及Zipkin和HTrace操作，为SpringCloud应用实现了一种分布式追踪解决方案。

### Spring Cloud Data Flow

- Data flow 是一个用于开发和执行大范围数据处理其模式包括ETL，批量运算和持续运算的统一编程模型和托管服务。

- 对于在现代运行环境中可组合的微服务程序来说，Spring Cloud data flow是一个原生云可编配的服务。使用Spring Cloud data flow，开发者可以为像数据抽取，实时分析，和数据导入/导出这种常见用例创建和编配数据通道 （data pipelines）。

- Spring Cloud data flow 是基于原生云对 spring XD的重新设计，该项目目标是简化大数据应用的开发。Spring XD 的流处理和批处理模块的重构分别是基于 Spring Boot的stream 和 task/batch 的微服务程序。这些程序现在都是自动部署单元而且他们原生的支持像 Cloud Foundry、Apache YARN、Apache Mesos和Kubernetes 等现代运行环境。

- Spring Cloud data flow 为基于微服务的分布式流处理和批处理数据通道提供了一系列模型和最佳实践。

### Spring Cloud Stream

Spring Cloud Stream是创建消息驱动微服务应用的框架。Spring Cloud Stream是基于Spring Boot创建，用来建立单独的／工业级spring应用，使用spring integration提供与消息代理之间的连接。数据流操作开发包，封装了与Redis,Rabbit、Kafka等发送接收消息。

### Spring Cloud Task

Spring Cloud Task 主要解决短命微服务的任务管理，任务调度的工作，比如说某些定时任务晚上就跑一次，或者某项数据分析临时就跑几次。

### Spring Cloud Zookeeper

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

### Spring Cloud Connectors

Spring Cloud Connectors 简化了连接到服务的过程和从云平台获取操作的过程，有很强的扩展性，可以利用Spring Cloud Connectors来构建你自己的云平台。

### Spring Cloud Starters

Spring Boot式的启动项目，为Spring Cloud提供开箱即用的依赖管理。

### Spring Cloud CLI

基于 Spring Boot CLI，可以让你以命令行方式快速建立云组件。

