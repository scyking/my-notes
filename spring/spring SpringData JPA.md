# spring SpringData JPA

标签： spring 

---

## 概述

- SpringData JPA

基于 JPA 规范开发的 SpringData 子模块，简化对数据的访问和操作。

- JPA 规范

全称 Java Persistence API，通过注解或XML描述“对象－关系表”的映射关系，并将运行期的实体对象持久化到数据库中。

## 优缺点

- 优点
    - 单表操作方便
- 缺点
    - 复杂查询构建繁琐
    - SQL优化困难

## SpringBoot 集成 SpringData JPA

1. 添加依赖

    ```
    <!-- 引入 SpringData JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- 引入 mysql -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    ```

2. 添加配置

    ```
    spring:
      datasource:
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/spring-boot-demo
        username: spring
        password: spring
    
      jpa:
        database: mysql
        show-sql: true
        hibernate:
          ddl-auto: update
    ```

3. CRUD 操作

    - 添加实体类，映射数据库表

    - 根据需求，实现 `Repository` 接口或其子集完成表操作
        - 常用接口
        - 方法命名规范


## 实现原理

基于 Java 动态代理。

springboot自动装配

1. 获取所有的 `Repository` 持久类

JpaRepositoriesAutoConfiguration
JpaRepositoriesAutoConfigureRegistrar--》ImportBeanDefinitionRegistrar --》registerBeanDefinitions（）方法是是Spring提供的策略,spring会在加载BeanDefinition的时候调用它的方法
RepositoryConfigurationDelegate --》registerRepositoriesIn（）方法注册Repositories持久类--》getRepositoryConfigurations（）获取Repository的配置信息，（包含扫描包获取类和区分Repository类）--》getCandidates（）方法扫描所有的class
到此,第一步完成,我们知道了Spring Data JPA如何通过包扫描获得哪些类需要被代理.

1. 注册 `FactoryBean` 的过程，注册到spring容器中。就是JpaRepositoryFactoryBean。

1. Spring会根据我们注册的BeanDefinition实例化JpaRepositoryFactoryBean,并调用它的getObject方法获得实际的Repository代理。
JpaRepositoryFactoryBean --》afterPropertiesSet()--》getRepository（）设置代理对象（打断点调试）--》QueryExecutorMethodInterceptor（）方法用于根据规则自动生成代码
