# spring 事务

标签（空格分隔）： spring

---

[toc]

## 介绍
事务管理在系统开发中是不可缺少的一部分，Spring提供了很好事务管理机制，主要分为编程式事务和声明式事务两种。

- 编程式事务：是指在代码中手动的管理事务的提交、回滚等操作，代码侵入性比较强

    ```
    try {
        //TODO something
         transactionManager.commit(status);
    } catch (Exception e) {
        transactionManager.rollback(status);
        throw new InvoiceApplyException("异常失败");
    }
    ```
- 声明式事务：基于AOP面向切面的，它将具体业务与事务处理部分解耦，代码侵入性很低

    ```
    @Transactional
    @GetMapping("/test")
    public String test() {
        int insert = cityInfoDictMapper.insert(cityInfoDict);
    }
    ```
## `@Transactional` 注解

### 作用域

- 作用于类：当把`@Transactional` 注解放在类上时，表示所有该类的public方法都配置相同的事务属性信息。
- 作用于方法：当类配置了`@Transactional`，方法也配置了`@Transactional`，方法的事务会覆盖类的事务配置信息。
- 作用于接口：不推荐。接口上配置`Spring AOP`, 使用CGLib动态代理，会导致`@Transactional`注解失效。

### 属性

#### 事务传播行为
> `propagation` 代表事务的传播行为，默认值为 Propagation.REQUIRED，其他的属性信息如下：
    
1. `Propagation.REQUIRED`：如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。( 也就是说如果A方法和B方法都添加了注解，在默认传播模式下，A方法内部调用B方法，会把两个方法的事务合并为一个事务 ）
1. `Propagation.SUPPORTS`：如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。
1. `Propagation.MANDATORY`：如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常。
1. `Propagation.REQUIRES_NEW`：重新创建一个新的事务，如果当前存在事务，暂停当前的事务。( 当类A中的 a 方法用默认Propagation.REQUIRED模式，类B中的 b方法加上采用 Propagation.REQUIRES_NEW模式，然后在 a 方法中调用 b方法操作数据库，然而 a方法抛出异常后，b方法并没有进行回滚，因为Propagation.REQUIRES_NEW会暂停 a方法的事务 )
1. `Propagation.NOT_SUPPORTED`：以非事务的方式运行，如果当前存在事务，暂停当前的事务。
1. `Propagation.NEVER`：以非事务的方式运行，如果当前存在事务，则抛出异常。
1. `Propagation.NESTED` ：和 Propagation.REQUIRED 效果一样。

#### 事务隔离级别
> `isolation` 事务的隔离级别，默认值为 Isolation.DEFAULT。
   
1. `TransactionDefinition.ISOLATION_DEFAULT`: 使用后端数据库默认的隔离级别，Mysql 默认采用的 `REPEATABLE_READ`，隔离级别 Oracle 默认采用的 `READ_COMMITTED` 隔离级别。
1. `TransactionDefinition.ISOLATION_READ_UNCOMMITTED`: 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
1. `TransactionDefinition.ISOLATION_READ_COMMITTED`: 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
1. `TransactionDefinition.ISOLATION_REPEATABLE_READ`: 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
1. `TransactionDefinition.ISOLATION_SERIALIZABLE`: 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

#### 超时时间
> `timeout` 事务的超时时间，默认值为 -1。如果超过该时间限制但事务还没有完成，则自动回滚事务。

#### 只读事务
> `readOnly` 指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。

#### 触发事务回滚异常
> `rollbackFor` 用于指定能够触发事务回滚的异常类型，可以指定多个异常类型。

#### 抛出异常类型
> `noRollbackFor` 抛出指定的异常类型，不回滚事务，也可以指定多个异常类型。

## `@Transactional` 失效场景

### `@Transactional`应用在非`public`修饰的方法上
在Spring AOP代理时，`TransactionInterceptor` （事务拦截器）在目标方法执行前后进行拦截，`DynamicAdvisedInterceptor`（CglibAopProxy 的内部类）的 intercept 方法或 `JdkDynamicAopProxy` 的 invoke 方法会间接调用 `AbstractFallbackTransactionAttributeSource`的 `computeTransactionAttribute` 方法，获取`Transactional` 注解的事务配置信息。

    ```
    protected TransactionAttribute computeTransactionAttribute(Method method,
        Class<?> targetClass) {
            // Don't allow no-public methods as required.
            if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
            return null;
    }
    ```
此方法会检查目标方法的修饰符是否为`public`，不是`public`则不会获取`@Transactional`的属性配置信息。

### `@Transactional` 注解属性 `propagation` 设置错误

若是错误的配置以下三种 `propagation`，事务将不会发生回滚:

- `TransactionDefinition.PROPAGATION_SUPPORTS`：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- `TransactionDefinition.PROPAGATION_NOT_SUPPORTED`：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- `TransactionDefinition.PROPAGATION_NEVER`：以非事务方式运行，如果当前存在事务，则抛出异常。

### `@Transactional` 注解属性 `rollbackFor` 设置错误
`rollbackFor` 可以指定能够触发事务回滚的异常类型。Spring默认抛出了未检查unchecked异常（继承自 RuntimeException 的异常）或者 Error才回滚事务；其他异常不会触发回滚事务。如果在事务中抛出其他类型的异常，但却期望 Spring 能够回滚事务，就需要指定 rollbackFor属性。

```
// 希望自定义的异常可以进行回滚
@Transactional(propagation= Propagation.REQUIRED,rollbackFor= MyException.class
```
若在目标方法中抛出的异常是 rollbackFor 指定的异常的子类，事务同样会回滚。Spring 源码如下：

```
private int getDepth(Class<?> exceptionClass, int depth) {
        if (exceptionClass.getName().contains(this.exceptionName)) {
            // Found it!
            return depth;
}
        // If we've gone as far as we can go and haven't found it...
        if (exceptionClass == Throwable.class) {
            return -1;
}
return getDepth(exceptionClass.getSuperclass(), depth + 1);
}
```
### 同一个类中方法调用，导致`@Transactional`失效
有一个类Test，它的一个方法A，A再调用本类的方法B（不论方法B是用public还是private修饰），但方法A没有声明注解事务，而B方法有。则外部调用方法A之后，方法B的事务是不会起作用的。这是由于使用Spring AOP代理造成的，因为只有当事务方法被当前类以外的代码调用时，才会由Spring生成的代理对象来管理。

```
//@Transactional
@GetMapping("/test")
private Integer A() throws Exception {
    CityInfoDict cityInfoDict = new CityInfoDict();
    cityInfoDict.setCityName("2");
    /**
     * B 插入字段为 3的数据
     */
    this.insertB();
    /**
     * A 插入字段为 2的数据
     */
    int insert = cityInfoDictMapper.insert(cityInfoDict);

    return insert;
}

@Transactional()
public Integer insertB() throws Exception {
    CityInfoDict cityInfoDict = new CityInfoDict();
    cityInfoDict.setCityName("3");
    cityInfoDict.setParentCityId(3);

    return cityInfoDictMapper.insert(cityInfoDict);
}
```
### 异常被`catch`，导致`@Transactional`失效

```
@Transactional
private Integer A() throws Exception {
    int insert = 0;
    try {
        CityInfoDict cityInfoDict = new CityInfoDict();
        cityInfoDict.setCityName("2");
        cityInfoDict.setParentCityId(2);
        /**
         * A 插入字段为 2的数据
         */
        insert = cityInfoDictMapper.insert(cityInfoDict);
        /**
         * B 插入字段为 3的数据
         */
        b.insertB();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
如果B方法内部抛了异常，而A方法此时try catch了B方法的异常，事务不能回滚。会抛出异常：
```
org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only
```
因为当ServiceB中抛出了一个异常以后，ServiceB标识当前事务需要rollback。但是ServiceA中由于你手动的捕获这个异常并进行处理，ServiceA认为当前事务应该正常commit。此时就出现了前后不一致，也就是因为这样，抛出了前面的`UnexpectedRollbackException`异常。

### 数据库引擎不支持事务
事务能否生效数据库引擎是否支持事务是关键。常用的MySQL数据库默认使用支持事务的`innodb`引擎。一旦数据库引擎切换成不支持事务的`myisam`，那事务就从根本上失效了。
