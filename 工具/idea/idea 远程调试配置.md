# idea 远程调试配置

标签（空格分隔）： idea

---

## 修改 maven 配置

`pom.xml`文件中添加如下配置：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <jvmArguments>
                    -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=8018
                </jvmArguments>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 添加 IDEA 配置

<img src="/md/idea/202141516294681.png" width="50%" alt="配置示例">

## 启动步骤

- 发布jar包`XXXXXX.jar`
- 执行如下命令启动：

```bash
java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8018,suspend=n -jar XXXXXX.jar
```

PS:`address`配置值需与maven中配置保持一致,且不与其他服务冲突，建议服务端口号-1000取值。
