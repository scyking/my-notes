# Swagger3

标签（空格分隔）： swagger spring-boot

---

[toc]

## SpringFox 3.0相关特性

- 与2.0更好的规范兼容性
- 支持`OpenApi 3.0.3`
- 轻依赖 `spring-plugin`，`swagger-core`
- 支持 `Spring 5`，`Webflux`（仅请求映射支持，尚不支持功能端点）、`Spring Integration`
- 补充官方在 spring boot 的自动装配 `pringfox-boot-starter` (以后可以直接依赖一个 dependency)

## SpringDoc 3.0相关特性

- 支持`Oauth2`登录
- 不兼容2.0

## 注解对比

使用`SpringDoc`版需使用`OpenAPI 3`注解，因其不向上兼容。

|swagger2|OpenAPI 3|注解位置
|--
|@Api|@Tag(name = “接口类描述”)|Controller 类上
|@ApiOperation|@Operation(summary =“接口方法描述”)|	Controller 方法上
|@ApiImplicitParams	|@Parameters|Controller 方法上
|@ApiImplicitParam	|@Parameter(description=“参数描述”)	|Controller 方法上 @Parameters 里
|@ApiParam	|@Parameter(description=“参数描述”)|Controller 方法的参数上
|@ApiIgnore	|@Parameter(hidden = true)/@Operation(hidden = true)/@Hidden|	-
|@ApiModel|	@Schema|DTO类上
|@ApiModelProperty|	@Schema|DTO属性上

## 使用(SpringFox版本)

1. 添加依赖
    ```
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-boot-starter</artifactId>
        <version>3.0.0</version>
    </dependency>
    ```

1. 配置文件

    ```
    # 生产环境需设置为false
    springfox.documentation.swagger-ui.enabled=true
    ```






