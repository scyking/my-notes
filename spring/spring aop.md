# spring aop

标签（空格分隔）： spring

---
[toc]

## 在pom中添加依赖
```
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-aop</artifactId>
	</dependency>
```
## 目标类
```
	@RestController
	public class StudentController {

    	@GetMapping(value = "/aoptest")
    	public String aopTest(){
        	return " AOP test success!";
    	}
	}
```
## 切面类
```
	@Aspect
	@Configuration
	public class HttpAspect {

    	//使用org.slf4j.Logger,这是Spring实现日志的方法
    	private final static Logger logger = LoggerFactory.getLogger(HttpAspect.class);

    	/**
     	* 定义AOP扫描路径
     	*/
    	@Pointcut("execution(public * com.example.myproject.controller.StudentController.*())")
    	public void log() {
    	}

    	/**
     	* 记录HTTP请求开始时的日志
     	*/
    	@Before("log()")
    	public void doBefore(JoinPoint joinPoint) {
        	ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        	HttpServletRequest request = attributes.getRequest();
        	//URL
        	logger.info("url={}", request.getRequestURI());
        	//method
        	logger.info("method={}", request.getMethod());
        	//ip
        	logger.info("ip={}", request.getRemoteAddr());
        	//类方法
        	logger.info("class={} and method name = {}", joinPoint.getSignature().getDeclaringTypeName(), joinPoint.getSignature().getName());
        	//参数
        	logger.info("参数={}", joinPoint.getArgs());
    	}

    	/**
     	* 记录HTTP请求结束时的日志
     	*/
    	@After("log()")
    	public void doAfter() {
        	ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        	HttpServletRequest request = attributes.getRequest();
        	logger.info("url = {} end of execution", request.getRequestURL());
    	}

    	/**
     	* 获取返回内容
     	*
     	* @param object
     	*/
    	@AfterReturning(returning = "object", pointcut = "log()")
    	public void doAfterReturn(Object object) {
        	logger.info("response={}", object.toString());
    	}
	}
```
PS:`Spring`通过`aop.xml`中`<aop:config>`配置目标对象、切入点等之间的联系。示例配置如下：
```
	<aop:config>
		<aop:aspect id="log" ref="HttpAspect">
        	<aop:pointcut id="printLog" expression="execution(* com.example.myproject.controller.StudentController.*())" />
        	<aop:before method="doBefore" pointcut-ref="printLog" />
        	<aop:after method="doAfter" pointcut-ref="printLog" />
        </aop:aspect>
    </aop:config>
```
### 注解解释
- `@Aspect`：描述一个切面类。
- `@Configuration`：描述一个配置类。
- `@Pointcut`：声明一个切入点，切入点决定了连接点关注的内容，可控制通知什么时候执行。
- `@Before`：前置通知。在某连接点之前执行的通知，但这个通知不能阻止连接点之前的执行流程（除非它抛出一个异常）。
- `@After` 最终通知。当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。
- `@AfterReturning`：后置通知。在某连接点正常完成后执行的通知，通常在一个匹配的方法返回的时候执行。
### 切入点表达式
格式如下：
```
execution([可见性] 返回类型 [声明类型].方法名(参数) [异常])
```

- `[]` :表示可选。
- 通配符:
    - `*`：匹配所有字符.
    - `..`：一般用于匹配多个包，多个参数.
    - `+`：表示类及其子类.
- 运算符：`&&`、`||`、`!`
	
## 启动类
```
	@SpringBootApplication
	public class MyprojectApplication extends SpringBootServletInitializer {

    	@Override
    	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        	return application.sources(MyprojectApplication.class);
    	}

    	public static void main(String[] args) {

        	SpringApplication.run(MyprojectApplication.class, args);
    	}
	}
```
<font color='RED'>PS: `spring-boot` 默认以启动类所在包为根路径，扫描当前包及子包。</font>




