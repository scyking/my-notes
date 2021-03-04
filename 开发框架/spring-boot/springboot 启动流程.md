# SpringBoot启动流程

标签（空格分隔）： spring-boot

---
[TOC]

https://www.cnblogs.com/theRhyme/p/11057233.html

## 整体流程图
![启动结构图](https://www.processon.com/view/link/59812124e4b0de2518b32b6e)

## 启动类

程序启动从标注`@SpringBootApplication`注解的`main`方法开始
```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
包含两部分:`@SpringBootApplication`，`SpringApplication.run(Application.class, args)`。

## `@SpringBootApplication`
```
@Target(ElementType.TYPE) // 注解的适用范围，其中TYPE用于描述类、接口（包括包注解类型）或enum声明
@Retention(RetentionPolicy.RUNTIME) // 注解的生命周期，保留到class文件中（三个生命周期）
@Documented // 表明这个注解应该被javadoc记录
@Inherited // 子类可以继承该注解
@SpringBootConfiguration // 继承了Configuration，表示当前是注解类
@EnableAutoConfiguration // 开启springboot的注解功能，springboot的四大神器之一，其借助@import的帮助
@ComponentScan(excludeFilters = { // 扫描路径设置（具体使用待确认）
@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
...
}
```

### `@SpringBootConfiguration`
继承了Configuration，表示当前是注解类，会被扫描加载到IOC容器。

1. 与`xml`配置`bean`方式区别
    - `xml`
    ```
    <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
default-lazy-init="true">
<!--bean定义-->
</beans>
    ```
        
    - `@Configuration`
    ```
    @Configuration
public class MockConfiguration{
    //bean定义
}
    ```
1. 与`xml`注入`bean`方式区别
    - `xml`
    ```
    <bean id="mockService" class="..MockServiceImpl">
...
</bean>
    ```
    
    - `@Configuration`
    ```
    @Configuration
public class MockConfiguration{
    @Bean
    public MockService mockService(){
        return new MockServiceImpl();
    }
}
    ```
> 任何一个标注了`@Bean`的方法，其**返回值**将作为一个bean定义注册到Spring的IoC容器，**方法名**将默认成该bean定义的id。
    
1. 与`xml`表达`bean`之间依赖关系区别
    - `xml`
    ```
    <bean id="mockService" class="..MockServiceImpl">
　　<propery name ="dependencyService" ref="dependencyService" />
</bean>
<bean id="dependencyService" class="DependencyServiceImpl"></bean>
    ```
    
    - `@Configuration`（**重点**）
    ```
    @Configuration
    public class MockConfiguration{
　　@Bean
　　public MockService mockService(){
        return new MockServiceImpl(dependencyService());
　　}
　　@Bean
　　public DependencyService dependencyService(){
    　　return new DependencyServiceImpl();
　　}
}
    ```
> 如果一个bean A的定义依赖其他bean B,则直接调用对应的JavaConfig类中依赖bean B的创建方法就可以了。
        
### `@ComponentScan`
> 扫描并加载默认包或指定包下面符合条件的组件。

- 作用
    1. 对应`xml`配置中的元素；
    1. （重点）`ComponentScan`的功能其实就是自动扫描并加载符合条件的组件（比如`@Component`和`@Repository`等）或者bean定义;
    1. 将这些bean定义加载到IoC容器中.

> 可以通过`basePackages`等属性来细粒度的定制`@ComponentScan`自动扫描的范围，如果不指定，则默认Spring框架实现会从声明`@ComponentScan`所在类的package进行扫描。
> 所以SpringBoot的启动类最好是放在root package下，因为默认不指定basePackages

### `@EnableAutoConfiguration`（**重点**）
> 开启springboot的注解功能（自动配置）。

在spring框架中就提供了各种以`@Enable`开头的注解，借助`@Import`的支持，收集和注册特定场景相关的bean定义。例如： `@EnableScheduling`、`@EnableCaching`、`@EnableMBeanExport`等。　　

- `@EnableScheduling`是通过`@Import`将Spring调度框架相关的bean定义都加载到IoC容器【定时任务、时间调度任务】
- `@EnableMBeanExport`是通过`@Import`将JMX相关的bean定义加载到IoC容器【监控JVM运行时状态】
- `@EnableAutoConfiguration`也是借助`@Import`的帮助，将所有符合自动配置条件的bean定义加载到IoC容器。

```
@SuppressWarnings("deprecation")
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage【重点注解】
@Import(AutoConfigurationImportSelector.class)【重点注解】
public @interface EnableAutoConfiguration {
...
}
```

1. `@AutoConfigurationPackage`
    ```
    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @Import(AutoConfigurationPackages.Registrar.class)
    public @interface AutoConfigurationPackage {
    ...
    }
    ```
    通过`@Import(AutoConfigurationPackages.Registrar.class)`，注册了一个Bean的定义。
    ```
    static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
 
        @Override
        public void registerBeanDefinitions(AnnotationMetadata metadata,
                BeanDefinitionRegistry registry) {
            register(registry, new PackageImport(metadata).getPackageName());
        }
        ...
    }
    ```
    `new PackageImport(metadata)`，它返回了当前主程序类的同级以及子级的包组件（重点）；
    
1. `@Import(AutoConfigurationImportSelector.class)`

    `AutoConfigurationImportSelector.class` 实现了 `DeferredImportSelector` 从 `ImportSelector`继承的方法`selectImports()`
    ```
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        }
        AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
                .loadMetadata(this.beanClassLoader);
        AnnotationAttributes attributes = getAttributes(annotationMetadata);
        List<String> configurations = getCandidateConfigurations(annotationMetadata,
                attributes);
        configurations = removeDuplicates(configurations);
        Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        configurations = filter(configurations, autoConfigurationMetadata);
        fireAutoConfigurationImportEvents(configurations, exclusions);
        return StringUtils.toStringArray(configurations);
    }
    ```
    第9行代码是加载各个组件jar下的` "META-INF/spring.factories"`外部文件。
    该方法在springboot启动流程——bean实例化前被执行，返回要实例化的类信息列表；
    如果获取到类信息，spring可以通过类加载器将类加载到jvm中，现在我们已经通过spring-boot的starter依赖方式依赖了我们需要的组件，那么这些组件的类信息在select方法中就可以被获取到。
    
    ```
    protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
 }
    ```
    其返回一个自动配置类的类名列表，方法调用了`loadFactoryNames()`。
    ```
    public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
        String factoryClassName = factoryClass.getName();
        return (List)loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
    }
    ```
    自动配置器会跟根据传入的`factoryClass.getName()`到项目系统路径下所有的`spring.factories`文件中找到相应的key，从而加载里面的类。
    
1. `SpringFactoriesLoader`详解
    提供一种配置查找的功能支持，配合`@EnableAutoConfiguration`使用，实现自动配置。
    ```
    public abstract class SpringFactoriesLoader {
    
        public static <T> List<T> loadFactories(Class<T> factoryClass, ClassLoader classLoader) {
        ...
        }
        /**
        * 方法会根据指定的classLoader，加载该类加器搜索路径下的指定文件，即spring.factories文件
        * @param factoryClass 工厂类名称
        * @param classLoader 对应的类加载器
        **/
        public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
        ...
        }
    }
    ```
    从`classpath`中搜寻所有的`META-INF/spring.factories`配置文件，并将其中`org.springframework.boot.autoconfigure.EnableAutoConfiguration`对应的配置项通过**反射**实例化为对应的标注了`@Configuration`的 JavaConfig 形式的 IoC 容器配置类，然后汇总为一个并加载到 IoC 容器。
    
## `SpringApplication`执行流程

### 整体流程图

```flow
st=>start: 开始
e=>end: 结束
op1=>operation: 执行@SpringBootApplication
执行SpringApplication.run()
初始化SpringApplication对象
收集各种条件和回调接口
op2=>operation: 创建并准备Environment
op3=>operation: 创建并初始化ApplicationContext
（使用SpringFactoriesLoader
查找并加载classpath中
所有可用的ApplicationContextInitializer
将之前通过@EnableAutoConfiguration
获取的配置加载到ApplicationContext）
op4=>operation: refresh ApplicationContext
    
sub1=>subroutine: 通告started()
sub2=>subroutine: 通告environmentPrepared()
sub3=>subroutine: 1. 通告contextPrepared()
2. 通告contextLoaded()
sub4=>subroutine: 1. 执行CommanadLineRunner
2. 通告finished()
c=>condition: showBanner==true
op5=>operation: 打印SpringBoot横幅
    
st->op1
op1->sub1
sub1->op2->sub2
sub2->c
c(no)->op3->sub3
c(yes)->op5->op3
sub3->op4->sub4
sub4->e
```
EventPublishingRunListener实现了SpringApplicationRunListener接口。
    
```
public interface SpringApplicationRunListener {
    default void starting() {}
    default void environmentPrepared(ConfigurableEnvironment environment) {}
    default void contextPrepared(ConfigurableApplicationContext context) {}
    default void contextLoaded(ConfigurableApplicationContext context) {}
    default void started(ConfigurableApplicationContext context) {}
    default void running(ConfigurableApplicationContext context) {}
    default void failed(ConfigurableApplicationContext context, Throwable exception) {}
}
```

### 整体流程

1. 创建配置环境(environment)
2. 创建事件监听(listners)
3. 创建应用上下文(applicationContext)
4. 基于以上条件，在容器中实例化bean。


    
    
    







