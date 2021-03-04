# spring 常用注解

标签（空格分隔）： spring

---

[toc]

## 声明bean注解
- `@Component` 组件，没有明确的角色
- `@Service` 在业务逻辑层使用（service层）。与`@Component`相比更加语义化
- `@Repository` 在数据访问层使用（dao层）。与`@Component`相比更加语义化
- `@Controller` 在展现层使用，控制器的声明（C）。与`@Component`相比更加语义化
- `@Configuration` 声明当前类为配置类。与`@Component`相比更加语义化
- `@Bean` 注解在方法上，声明当前方法的返回值为一个bean，替代xml中的方式（方法上）
- `@ComponentScan` 用于对Component进行扫描，相当于xml中的（类上）
- `@WishlyConfiguration` 为`@Configuration`与`@ComponentScan`的组合注解，可以替代这两个注解

## 注入bean的注解
- `@Autowired`：由Spring提供，默认根据类型自动装配。配合`@Qualifier`指定bean名称
- `@Inject`：由JSR-330提供
- `@Resource`：由JSR-250提供，默认根据id（name）装配


## 切面（AOP）相关注解
在java配置类中使用`@EnableAspectJAutoProxy`注解开启Spring对AspectJ代理的支持（类上）

- `@Aspect` 声明一个切面（类上）
- `@After` 在方法执行之后执行（方法上）
- `@Before` 在方法执行之前执行（方法上）
- `@Around` 在方法执行之前与之后执行（方法上）
- `@PointCut` 声明切点


## @Bean的属性支持
- `@Scope` 设置Spring容器如何新建Bean实例（方法上，得有@Bean）
其设置类型包括：
    - `Singleton` （单例,一个Spring容器中只有一个bean实例，默认模式）,
    - `Protetype` （每次调用新建一个bean）,
    - `Request` （web项目中，给每个http request新建一个bean）,
    - `Session` （web项目中，给每个http session新建一个bean）,
    - `GlobalSession`（给每一个 global http session新建一个Bean实例）
- `@StepScope` 在Spring Batch中还有涉及
- `@PostConstruct` 由JSR-250提供，在构造函数执行完之后执行，等价于xml配置文件中bean的initMethod
- `@PreDestory` 由JSR-250提供，在Bean销毁之前执行，等价于xml配置文件中bean的destroyMethod

## @Value
为属性注入值（属性上）。支持方式：

- 注入普通字符

    ```
    @Value("Jackson")
    String name;
    ```
- 注入操作系统属性

    ```
    @Value("{systemProperties['os.name']}")
    String osName;
    ```
- 注入表达式结果

    ```
    @Value("#{ T(java.lang.Math).random() * 100}")
    String randomNumber;
    ```
- 注入其他bean属性

    ```
    @Value("#{domeClass.name}")
    String name;
    ```
- 注入文件资源

    ```
    @Value("classpath:com/hgs/hello/test.txt")
    String Resource file;
    ```
- 注入网站资源

    ```
    @Value("http://www.baidu.com)
    Resource url;
    ```
- 注入配置文件

    ```
    @Value("${book.name}")
    String name;
    ```

## @ConfigurationProperties
通过@ConfigurationProperties读取配置信息并与 bean 绑定。

```
@Data
@Component
@ConfigurationProperties(prefix = "library")
class LibraryProperties {
    private String location;
    private List<Book> books;

    @Data
    static class Book {
        String name;
        String description;
    }
}
```
配置文件：
```
library:
  location: 湖北武汉加油中国加油
  books:
    - name: 天才基本法
      description: 略。
    - name: 时间的秩序
      description: 略。
    - name: 了不起的我
      description: 略。
```

## 环境切换
- `@Profile` 通过设定Environment的ActiveProfiles来设定当前context需要使用的配置环境。（类或方法上）
- `@Conditional` Spring4中可以使用此注解定义条件话的bean，通过实现Condition接口，并重写matches方法，从而决定该bean是否被实例化。（方法上）

## 异步相关
- `@EnableAsync` 配置类中，通过此注解开启对异步任务的支持，叙事性AsyncConfigurer接口（类上）
- `@Async` 在实际执行的bean方法使用该注解来申明其是一个异步任务（方法上或类上所有的方法都将异步，需要`@EnableAsync`开启异步任务）

## 定时任务相关
- `@EnableScheduling` 在配置类上使用，开启计划任务的支持（类上）
- `@Scheduled` 来申明这是一个任务，包括cron,fixDelay,fixRate等类型（方法上，需先开启计划任务的支持）

## @Enable 注解说明
这些注解主要用来开启对xxx的支持。

- `@EnableAspectJAutoProxy` 开启对AspectJ自动代理的支持
- `@EnableAsync` 开启异步方法的支持
- `@EnableScheduling` 开启计划任务的支持
- `@EnableWebMvc` 开启Web MVC的配置支持
- `@EnableConfigurationProperties` 开启`对@ConfigurationProperties`注解配置Bean的支持
- `@EnableJpaRepositories` 开启对SpringData JPA Repository的支持
- `@EnableTransactionManagement` 开启注解式事务的支持
- `@EnableCaching` 开启注解式的缓存支持

## SpringMVC部分
- `@EnableWebMvc` 在配置类中开启Web MVC的配置支持，如一些`ViewResolver`或者`MessageConverter`等，若无此句，重写`WebMvcConfigurerAdapter`方法（用于对SpringMVC的配置）。

- `@Controller` 声明该类为SpringMVC中的Controller
- `@RequestMapping` 用于映射Web请求，包括访问路径和参数（类或方法上）
    - `@GetMapping` 等同于 `@RequestMapping(method = RequestMethod.GET)`
    - `@PostMapping` 等同于 `@RequestMapping(method = RequestMethod.POST)`
    - `@PutMapping` 等同于 `@RequestMapping(method = RequestMethod.PUT)`
    - `@DeleteMapping` 等同于 `@RequestMapping(method = RequestMethod.DELETE)`
    - `@PatchMapping` 等同于 `@RequestMapping(method = RequestMethod.PATCH)`
- `@ResponseBody` 支持将返回值放在response内，而不是一个页面，通常用户返回json数据（返回值旁或方法上）
- `@RequestBody` 允许request的参数在request体中，而不是在直接连接在地址后面。（放在参数前）
- `@PathVariable` 用于接收路径参数，比如`@RequestMapping(“/hello/{name}”)`申明的路径，将注解放在参数中前，即可获取该值，通常作为Restful的接口实现方法。
- `@RestController` 该注解为一个组合注解，相当于`@Controller`和`@ResponseBody`的组合，注解在类上，意味着，该Controller的所有方法都默认加上了`@ResponseBody`。
- `@ControllerAdvice` 通过该注解，我们可以将对于控制器的全局配置放置在同一个位置，注解了`@Controller`的类的方法可使用`@ExceptionHandler`、`@InitBinder`、`@ModelAttribute`注解到方法上，这对所有注解了`@RequestMapping`的控制器内的方法有效。
    - `@ExceptionHandler` 全局异常处理
    - `@InitBinder` 全局数据预处理
    - `@ModelAttribute` 全局数据绑定

## 参数校验
JSR(Java Specification Requests）、Hibernate Validator。所有的注解，推荐使用 JSR 注解，即`javax.validation.constraints`，而不是`org.hibernate.validator.constraints`。

- `@Valid`
- `@Validated`
- `@NotEmpty` 被注释的字符串的不能为 null 也不能为空
- `@NotBlank` 被注释的字符串非 null，并且必须包含一个非空白字符
- `@Null` 被注释的元素必须为 null
- `@NotNull` 被注释的元素必须不为 null
- `@AssertTrue` 被注释的元素必须为 true
- `@AssertFalse` 被注释的元素必须为 false
- `@Pattern(regex=,flag=)`被注释的元素必须符合指定的正则表达式
- `@Email` 被注释的元素必须是 Email 格式。
- `@Min(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@Max(value)`被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- `@DecimalMin(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@DecimalMax(value)` 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- `@Size(max=, min=)`被注释的元素的大小必须在指定的范围内
- `@Digits (integer, fraction)`被注释的元素必须是一个数字，其值必须在可接受的范围内
- `@Past`被注释的元素必须是一个过去的日期
- `@Future` 被注释的元素必须是一个将来的日期

## JPA相关

- `@Entity`声明一个类对应一个数据库实体
- `@Table` 设置表名
- `@Id` 声明一个字段为主键。
- `@GeneratedValue` 指定主键生成策略。配合`@Id`使用
- `@GenericGenerator` 声明一个主键策略
- `@Column` 声明字段。
- `@Transient` 声明不需要与数据库映射的字段，在保存的时候不需要保存进数据库 。
- `@Lob` 声明某个字段为大字段。
- `@Enumerated` 可以使用枚举类型的字段
- `@OneToOne` 声明一对一关系
- `@OneToMany` 声明一对多关系
- `@ManyToOne` 声明多对一关系
- `@ManyToMany` 声明多对多关系

## 事务@Transactional
> `Exception`分为运行时异常`RuntimeException`和非运行时异常。在`@Transactional`注解中如果不配置`rollbackFor`属性,那么事物只会在遇到`RuntimeException`的时候才会回滚,加上`rollbackFor=Exception.class`,可以让事物在遇到非运行时异常时也回滚。

`@Transactional` 注解一般用在可以作用在类或者方法上。

- 作用于类：当把`@Transactional` 注解放在类上时，表示所有该类的`public`方法都配置相同的事务属性信息。
- 作用于方法：当类配置了`@Transactional`，方法也配置了`@Transactional`，方法的事务会覆盖类的事务配置信息。

## json数据处理
- `@JsonIgnoreProperties` 作用在类上用于过滤掉特定字段不返回或者不解析。
- `@JsonIgnore` 一般用于类的属性上，作用和上面的@JsonIgnoreProperties 一样。
- `@JsonFormat` 一般用来格式化 json 数据。：    
- `@JsonUnwrapped` 扁平化对象




