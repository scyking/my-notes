# spring bean

标签（空格分隔）： spring

---

[toc]

## 
https://www.cnblogs.com/theRhyme/p/11057233.html#_label6
https://www.cnblogs.com/javazhiyin/p/10905294.html
https://www.cnblogs.com/zrtqsk/p/3735273.html


## 一、什么是 Spring Bean 

所有可以被spring容器实例化并管理的java类都可以称为bean。

- Spring Bean 相关注解分类

	- 使用Bean。即直接使用在xml文件中配置好的Bean，完成属性、方法的组装；比如`@Autowired`、 `@Resource`，可以通过`byTYPE(@Autowired)`、`byNAME(@Resource)`的方式获取Bean；

	- 注册Bean。`@Component`、`@Repository`、`@Controller`、`@Service`、`@Configration`，这些注解都是将实例化的对象转化成一个Bean，放在IoC容器中。在使用的时候，会和上面的`@Autowired`、`@Resource`注解相配合，将对象、属性、方法进行组装。

## 二、Spring Bean 作用域 

- `singleton`：单例。作用域默认是单例，这种范围确保不管接受到多少个请求，每个容器中只有一个bean的实例，单例的模式由bean factory自身来维护。

- `prototype`：原型。与单例范围相反，为每一个bean请求提供一个实例。

- `request`：请求。为每一个来自客户端的网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。

- `Session`：会话。与请求范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。

- `global-session`：全局。在一个全局的Http Session中，容器会返回该Bean的同一个实例，仅在使用portlet context时有效。

## 三、Spring Bean 生命周期

- Spring BeanFactory 容器

	1. 容器启动
	2. 实例化bean对象
	3. 设置对象属性
	4. 调用`BeanNameAware`的`setBeanName()`方法
	5. 调用`BeanFactoryAware`的`setBeanFactory()`方法
	7. 调用`BeanPostProcessor`的预初始化方法`(before)`(必须手动注册)
	8. 调用`InitializingBean`的`afterPropertiesSet()`方法
	9. 调用`init-method`
	10. 调用`BeanPostProcessor`后初始化方法`(after)`（必须手动注册）
	11. 分以下两种情况
		- `scope`为`singleton`的缓存在`spring ioc`容器中
		- `scope`为`prototype`的生命周期交给客户端
	12. 容器关闭
	13. 调用`disposableBean`的`afterPropertiesSet()`方法
	14. 调用`destory-method`的方法

- Spring ApplicationContext 容器

	1. 容器启动
	2. 实例化bean对象
	3. 设置对象属性
	4. 调用`BeanNameAware`的`setBeanName()`方法
	5. 调用`BeanFactoryAware`的`setBeanFactory()`方法
	6. 调用`ApplicationContextAware`的`setApplicationContext()`方法
	7. 调用`BeanPostProcessor`的预初始化方法`(before)`
	8. 调用`InitializingBean`的`afterPropertiesSet()`方法
	9. 调用`init-method`
	10. 调用`BeanPostProcessor`后初始化方法`(after)`
	11. 分以下两种情况
		- `scope`为`singleton`的缓存在`spring ioc`容器中
		- `scope`为`prototype`的生命周期交给客户端
	12. 容器关闭
	13. 调用`disposableBean`的`afterPropertiesSet()`方法
	14. 调用`destory-method`的方法

- 两种容器重Bean生命周期比较

	1. BeanFactory容器中，不会调用`ApplicationContextAware`接口的`setApplicationContext()`方法

	2. BeanFactory容器中，`BeanPostProcessor`接口的`postProcessBeforeInitialization`方法和`postProcessAfterInitialization`方法不会自动调用，必须自己通过代码手动注册

	3. BeanFactory容器启动的时候，不会去实例化所有`bean`，包括所有scope为`singleton`且非延迟加载的`bean`也是一样，而是在调用的时候去实例化。

