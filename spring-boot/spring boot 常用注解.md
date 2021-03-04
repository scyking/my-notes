# spring boot 常用注解

标签（空格分隔）： spring-boot

---

## 常用注解

|名称|描述|
|:---:|:---:|
|`@ResponseBody`|使用此注解之后不会再走视图处理器，而是直接将数据写入到输入流中，他的效果等同于通过 `response` 对象输出指定格式的数据。|



## 常用注解比较

- `@Controller` 与 `@RestController`
- `@RequestMapping` 与 `@PostMapping`、`@GetMapping`
- `@PathVariable`、`@RequestParam`、`@RequestBody`、`@RequestHeader`、`@CookieValue`及无注解方式
- `HttpServletResponse`、`HttpServletRequest`

```
<!-- mybatis -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>

        <!-- mybatis-plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.0.6</version>
        </dependency>
```






