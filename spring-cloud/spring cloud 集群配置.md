# spring cloud 集群配置

标签（空格分隔）： spring-cloud

---

[toc]

## 服务端
```
eureka:
    client:
    #Eureka是为注册中心,是否需要将自己注册到注册中心上(默认为true),本次位单机部署,不需要设置为true;但是注册中心集群时候必须为true;因为集群时，其他Eureka需要从此Eureka中的节点上获取数据；
    register-with-eureka: false
    #Erueka是为注册中心，不需要检索服务信息;(表示是否从Eureka Server获取注册信息，默认为true。 如果这是一个单点的 Eureka Server，不需要同步其他节点的数据，可以设为false)
    fetch-registry: false
        serviceUrl:
          # 注册中心地址。集群地址以逗号隔开，且不包括本身地址。
          defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
    instance:
      hostname: localhost
```

## 客户端
- IP型
```
eureka:
  instance:
    statusPageUrlPath: /actuator/info
    healthCheckUrlPath: /actuator/health
    home-page-url-path: /
    ip-address: 120.xxx.xxx.xxx #决定Eureka服务显示host
    hostname: ${eureka.instance.ip-address} #决定服务跳转host，必须指定
    prefer-ip-address: false #不设可自己指定，true会自动获取本地ipv4地址
    instance-id: ${eureka.instance.ip-address}:${spring.application.name}:${server.port}
  client:
    serviceUrl:
      #            defaultZone: http://localhost:6600/eureka/
      defaultZone: http://120.xxx.xxx.xxx:6600/eureka/ #多注册中心以逗号隔开
```

- 常规型
```
eureka:
  instance:
    statusPageUrlPath: /actuator/info
    healthCheckUrlPath: /actuator/health
    home-page-url-path: /
  client:
    serviceUrl:
      defaultZone: http://120.xxx.xxx.xxx:6600/eureka/
```

## 注意事项
1. 常规型配置服务必须和网关路由分发在同一机器启动。如果全部为IP型配置则无此限制，在任意机器启动，注册到同一注册中心即可。 
2. 此处IP必须为可用的公网IP。



























