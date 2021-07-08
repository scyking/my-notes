# springboot启动后执行初始化方法

标签（空格分隔）： spring-boot

---

[toc]

## 方式

- 实现`CommandLineRunner`接口
- 实现`ApplicationRunner`接口
- 使用`@PostConstruct`注解

PS：顺序控制使用`@Order`注解。默认`ApplicationRunner`要比`CommandLineRunner`实现类先执行，`@PostConstruct`会在bean初始化前执行，方法不能依赖其它bean。

## 使用

- `CommandLineRunner`、`ApplicationRunner`实现类上添加`@Component`注解
- 在bean的方法上添加注解`@PostConstruct`


