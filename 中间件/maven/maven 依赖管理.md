# maven 依赖管理

标签（空格分隔）： maven

---

[toc]

## 坐标

- groupId：团体、公司、组织机构等等的唯一标识。
- artifactId：一个组织中项目的唯一标识。
- version：一个项目的版本。
- scope：表示依赖范围。

|依赖范围|编译有效|测试有效|运行时有效|打包有效|例子|
|---|:---:|:---:|:---:|:---:|---|
|complie|Y|Y|Y|Y|spring-core|
|test|N|Y|N|N|junit|
|provided|Y|Y|N|N|servlet-api|
|runtime|N|Y|Y|Y|JDBC驱动|
|system|Y|Y|N|N|本地maven仓库之外的类库|

## 依赖冲突

#### 产生原因

```
a ---> b ---> c 1.2
a ---> b ---> c 1.3
```

a依赖b，同时依赖d。a和b、d的关系是直接依赖关系，a和c的关系是间接依赖的关系。

#### 冲突解决

1. 先定义先使用
2. 路径最近原则

上面两条是默认行为，可通过手动控制排除依赖。

```
// 从 spring-context 中排除 spring-core 依赖。
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.9.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```