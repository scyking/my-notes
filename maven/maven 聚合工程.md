# maven 聚合工程

标签（空格分隔）：maven

---

> 所谓的聚合⼯程，实际上也就是多模块项⽬。

[TOC]

## 多模块项目展示

```
|--parent
    |-- ucenter
    |-- common
    |-- web
```

## 创建父项目

1. 创建 maven 项目，仅保留 `pom.xml` 文件，删除其他无关文件
2. `pom.xml` 主要内容如下：

    ```
    <!-- 项目说明 -->
    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>


    <!-- 模块说明 -->
    <modules>
        <module>ucenter</module>
        <module>common</module>
        <module>web</module>
    </modules>
    ```
    
## 创建子模块
> web 子模块举例。

1. 右键父项目，依次点击 `New` --> `Module` --> `Spring Initializr` 创建 spring boot 项目
2. `pom.xml` 内容如下：

    ```
    <!-- 基本信息 -->
    <groupId>com.example</groupId>
    <artifactId>web</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <!-- 继承父项目 -->
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent</artifactId>
        <version>1.0.0</version>
    </parent>
    ```
## 注意事项

1. 父模块使用`pom`打包方式，子模块使用`jar`打包方式。
1. 子模块不可直接单独打包，需从父模块处打包。