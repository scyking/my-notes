# 使用 idea 打包 jar

标签（空格分隔）： idea maven

---

> 使用命令：`java -jar XXX.jar`

[toc]

## java project 打包可执行jar

1. 项目上鼠标右键 --> `Open Module Settings`
2. 依次选择 `Artifacts` --> `+` --> `JAR` --> `From modules with dependencies...`
3. 选择主类 `Main Class`
4. 设置 `META-INF/MANIFEST.MF`，选择项目根目录下即可
5. 点击`ok`，回到工程目录（关闭弹框前，注意jar包保存位置）
6. 依次选择菜单栏 `Build` --> `Build Artifacts...` --> `XXX.jar` --> `Build`
7. 在项目根目录 `..\out\artifacts\..` 中找到相关jar包

## maven project 打包可执行jar

### 打无依赖jar包

1. 修改 `pom.xml` 文件，添加如下：

    ```
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>2.4</version>
        <configuration>
            <archive>
                <!-- 生成的jar中，是否包含pom.xml和pom.properties -->
                <addMavenDescriptor>false</addMavenDescriptor>
                <manifest>
                    <!-- 是否要把第三方jar放到manifest的classpath中-->
                    <addClasspath>true</addClasspath>
                    <!-- 生成的manifest中classpath的前缀，因为要把第三方jar放到lib目录下，所以classpath的前缀是lib/ -->
                    <classpathPrefix>lib/</classpathPrefix>
                    <!-- 应用的main class -->
                    <mainClass>com.scyking.demo.TestMain</mainClass>
                </manifest>
            </archive>
            <!-- 过滤掉不希望包含在jar中的文件 -->
            <excludes>
                <exclude>${project.basedir}/xml/*</exclude>
            </excludes>
        </configuration>
    </plugin>
    ```
1. 打开 idea maven 管理工具 `Maven Projects`
1. 点击相应项目下 `Lifecycle` --> `package`
2. 等待运行完成，会在项目`target`目录下生成相应jar包

### 打有依赖jar包

1. 修改 `pom.xml` 文件，添加如下：

    ```
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.1</version>
    </plugin>
    <plugin>
        <artifactId>maven-assembly-plugin</artifactId>
        <configuration>
            <archive>
                <manifest>
                    <mainClass>com.scyking.demo.TestMain</mainClass>
                </manifest>
            </archive>
            <descriptorRefs>
                <descriptorRef>jar-with-dependencies</descriptorRef>
            </descriptorRefs>
        </configuration>
        <!--下面是为了使用 mvn package 命令，如果不加则使用mvn assembly-->
        <executions>
            <execution>
                <id>make-assemble</id>
                <phase>package</phase>
                <goals>
                    <goal>single</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    ```
1. 打开 idea maven 管理工具 `Maven Projects`
1. 点击相应项目下 `Lifecycle` --> `package`
2. 等待运行完成，会在项目`target`目录下生成相应jar包

