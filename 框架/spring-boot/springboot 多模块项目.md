# springboot 多模块项目

标签（空格分隔）： spring-boot maven

---

> IDE 使用的 IDEA

[TOC]

## maven 坐标填写规范

- `groupId`：项目组织唯一标识。一般为多段，形式为 `域`.`公司名称`，对应 java 包结构
- `artifactId`：项目唯一标识。对应项目名称，即项目根目录名称
- `version`：版本信息
- `packaging`：打包方式

## 创建父项目

1. 创建 maven 项目，仅保留 `pom.xml` 文件，删除其他无关文件
2. `pom.xml` 内容如下：

    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!-- 基本信息 -->
    <description>Demo project for Spring Boot</description>
    <modelVersion>4.0.0</modelVersion>
    <name>spring-boot-demo</name>
    <packaging>pom</packaging>

    <!-- 项目说明 -->
    <groupId>com.example</groupId>
    <artifactId>spring-boot-demo</artifactId>
    <version>1.0.0</version>

    <!-- 继承说明：继承spring boot提供的父工程 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <!-- 模块说明 -->
    <modules>
        <module>ucenter</module>
        <module>common</module>
        <module>web</module>
    </modules>

    <!-- 版本说明：统一管理依赖的版本号 -->
    <dependencyManagement>
        <dependencies>
            <!-- 公共模块 -->
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>common</artifactId>
                <version>1.0.0</version>
            </dependency>
            <!-- 用户管理模块 -->
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>ucenter</artifactId>
                <version>1.0.0</version>
            </dependency>
            <!-- web 模块，包含主启动类 -->
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>web</artifactId>
                <version>1.0.0</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    </project>
    ```
    
## 创建子模块

### 创建 `web` 子模块

1. 右键父项目，依次点击 `New` --> `Module` --> `Spring Initializr` 创建 spring boot 项目
2. `pom.xml` 内容如下：

    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 基本信息 -->
    <groupId>com.example</groupId>
    <artifactId>web</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <name>web</name>

    <!-- 继承父项目 -->
    <parent>
        <artifactId>spring-boot-demo</artifactId>
        <groupId>com.example</groupId>
        <version>1.0.0</version>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <!-- web 模块相关依赖 -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- mysql 相关 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.1</version>
        </dependency>

        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- swagger -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.6.1</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.6.1</version>
        </dependency>

        <!--  异步httpclient -->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpasyncclient</artifactId>
            <version>4.1.1</version>
        </dependency>

        <!-- test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    </project>
    ```


### 创建 `ucenter` 子模块

1. 右键父项目，依次点击 `New` --> `Module` --> `Maven` 创建 maven 项目
2. `pom.xml` 内容如下：

    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 基本信息 -->
    <groupId>com.example</groupId>
    <artifactId>ucenter</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <name>ucenter</name>

    <!-- 继承父项目 -->
    <parent>
        <artifactId>spring-boot-demo</artifactId>
        <groupId>com.example</groupId>
        <version>1.0.0</version>
    </parent>

    <!-- ucenter 模块相关依赖 -->
    <dependencies>
    </dependencies>

    </project>
    ```

### 创建 `common` 子模块

`common` 及新增子模块创建方式均同 `ucenter` 子模块创建。

## 打包启动


https://blog.csdn.net/qq_32239767/article/details/79995231




