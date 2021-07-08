# MyBatis 中设计模式 

标签（空格分隔）： 设计模式 mybatis

---

[toc]

## 常用设计模式

|设计模式|举例|
|---|---|
|建造者模式|SqlSessionFactoryBuilder 、XMLConfigBuilder、XMLMapperBuilder、XMLStatementBuilder、CacheBuilder|
|工厂模式|SqlSessionFactory、ObjectFactory、MapperProxyFactory|
|单例模式|ErrorContext和LogFactory|
|**代理模式**|JDK:MapperProxy、ConnectionLogger；cglib:executor.loader包实现延迟加载|
|组合模式|SqlNode和各个⼦类|
|模板模式|BaseExecutor和SimpleExecutor|
|适配器模式|Log的Mybatis接⼝和它对jdbc、log4j等各种⽇志框架的适配实现|
|装饰者模式|Cache实现|
|迭代器模式|PropertyTokenizer实现了Iterator接口|

## 组合模式
> 组合模式组合多个对象形成树形结构以表⽰“整体-部分”的结构层次。
> 叶⼦对象和组合对象实现相同的接⼝，使叶⼦节点和对象节点能够进⾏⼀致处理。

### mybatis 动态sql实现

```
public interface SqlNode {
  boolean apply(DynamicContext context);
}
```

对于实现该接⼝的所有节点，就是整个组合模式树的各个节点：

```
SqlNode
|-- ChooseSqlNode
|-- ForEachSqlNode
|-- IfSqlNode
|-- MixedSqlNode
|-- StaticTextSqlNode
|-- TrimSqlNode
|   |-- SetSqlNode
|   |-- WhereSqlNode
|
|-- VarDeclSqlNode
```

## 模板模式
> 是基于继承的代码复用的基本技术。

### mybatis 执行sql实现
> `BaseExecutor`采用模板模式，实现大部分sql执行逻辑，然后把几个方法交给子类定制化完成。

```
protected abstract int doUpdate(MappedStatement ms, Object parameter) throws SQLException;

protected abstract List<BatchResult> doFlushStatements(boolean isRollback) throws SQLException;

protected abstract <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql)
      throws SQLException;

protected abstract <E> Cursor<E> doQueryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds, BoundSql boundSql)
      throws SQLException;
```

该模板⽅法类有⼏个⼦类的具体实现，使⽤了不同的策略：

- `SimpleExecutor`，每执⾏⼀次update或select，就开启⼀个Statement对象，⽤完⽴刻关闭Statement对象。
- `ReuseExecutor`，执⾏update或select，以sql作为key查找Statement对象，存在就使⽤，不存在就创建。⽤完后，不关闭Statement对象，⽽是放置于`Map<String, Statement>`内，供下⼀次使⽤。
- `BatchExecutor`，执⾏update（没有select，JDBC批处理不⽀持select），将所有sql都添加到批处理中（`addBatch()`），等待统⼀执⾏（`executeBatch()`）。

## 装饰者模式
> 动态地给⼀个对象增加⼀些额外的职责。

### mybatis 缓存实现
> 缓存功能由接口`Cache`定义。数据存储、缓存基本功能由`PerpetualCache`实现，然后通过装饰器进行控制。

`PerpetualCache` 装饰器：

1. `FifoCache`：先进先出算法，缓存回收策略
2. `LoggingCache`：输出缓存命中的⽇志信息
3. `LruCache`：最近最少使⽤算法，缓存回收策略
4. `ScheduledCache`：调度缓存，负责定时清空缓存
5. `SerializedCache`：缓存序列化和反序列化存储
6. `SoftCache`：基于软引⽤实现的缓存管理策略
7. `SynchronizedCache`：同步的缓存装饰器，⽤于防⽌多线程并发访问
8. `WeakCache`：基于弱引⽤实现的缓存管理策略
9. `TransactionalCache`：事务性缓存
10. `BlockingCache`：阻塞性缓存

缓存分类：

- ⼀级缓存，⼜叫本地缓存，是`PerpetualCache`类型的永久缓存，保存在执⾏器中（`BaseExecutor`），⽽执⾏器⼜在`SqlSession`中，所以⼀级缓存的⽣命周期与`SqlSession`是相同的。
- ⼆级缓存，⼜叫⾃定义缓存，实现了`Cache`接⼝的类都可以作为⼆级缓存。

## 迭代器模式
> 提供⼀种⽅法访问⼀个容器对象中各个元素，⽽⼜不需暴露该对象的内部细节。
> Java的`Iterator`就是迭代器模式的接⼝。只要实现了该接⼝，就相当于应⽤了迭代器模式。

