# MyBatis 事务管理

标签（空格分隔）： mybatis

---

[toc]

> ⽆论是`SqlSession`，还是`Executor`，它们的事务⽅法，最终都指向了`Transaction`的事务⽅法，即都是由`Transaction`来完成事务提交、回滚的。

## 事务接口 `Transaction`

```
public interface Transaction {
    Connection getConnection() throws SQLException;
    void commit() throws SQLException;
    void rollback() throws SQLException;                // 回滚当前未提交事务
    void close() throws SQLException;                   // 关闭⼀个conn连接，或者是把conn连接放回连接池内。（不是关闭事务）
    Integer getTimeout() throws SQLException;
}
```

### 实现类

- `JdbcTransaction`，单独使⽤Mybatis时，默认的事务管理实现类。是JDBC事务的极简封装，仅是让conn⽀持连接池。
- `ManagedTransaction`，含义为托管事务，空壳事务管理器。仅是提醒⽤⼾，把事务托管给其它框架（如Spring）。

## 事务工厂`TransactionFactory`

```
public interface TransactionFactory {
    default void setProperties(Properties props) {}
    Transaction newTransaction(Connection var1);
    Transaction newTransaction(DataSource var1, TransactionIsolationLevel var2, boolean var3);
}
```

### 实现类

- `JdbcTransactionFactory`
- `ManagedTransactionFactory`

## 特殊场景

1. 一个conn生命周期内，可以存在无数多个事务。
1. `autoCommit=false`，没有执行`commit()`，仅执行`close()`。
    > Mybatis会将事务进行`rollback`操作，再执行`conn.close()`。
2. `autoCommit=false`，没有执行`commit()`，也没有执行`close()`。
    > jvm结束前，如果事务隔离级别是`read uncommitted`，可以查到该条记录。jvm结束后，事务被`rollback()`，记录消失。
