# java 线程池拒绝策略

标签（空格分隔）： java-thread

---

[toc]

## 池化设计思想

> 通过初始预设资源，抵消每次获取资源的消耗。如，创建线程的开销、获取远程连接的开销等等。

### 特征

> 决定了池子拒绝策略触发的时机。

- 池子初始值
- 池子活跃值
- 池子最大值
- 等等

### 线程池拒绝策略触发时机

> 当前提交任务数⼤于（maxPoolSize + queueCapacity）时就会触发线程池的拒绝策略。

- `maxPoolSize`：线程池最大值
- `queueCapacity`：缓冲队列容量

## JDK内置的拒绝策略

### 拒绝策略接⼝定义

> 当触发拒绝策略时，线程池会将当前提交的任务以及线程池实例本⾝传递给设置好的具体策略处理。

```java
public interface RejectedExecutionHandler {
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

### 调⽤者运⾏策略（`CallerRunsPolicy`）

- 功能
    > 当触发拒绝策略时，只要线程池没有关闭，就由提交任务的当前线程处理。

- 使⽤场景
    > ⼀般在不允许失败的、对性能要求不⾼、并发量较⼩的场景下使⽤，因为线程池⼀般情况下不会关闭，也就是提交的任务⼀定会被运⾏。但是由于是调⽤者线程⾃⼰执⾏的，当多次提交任务时，就会阻塞后续任务执⾏，性能和效率⾃然就慢了。

- 代码

    ```java
    public static class CallerRunsPolicy implements RejectedExecutionHandler {
        public CallerRunsPolicy() {
        }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                r.run();
            }
        }
    }
    ```

### 中⽌策略（`AbortPolicy`）

- 功能
    > 当触发拒绝策略时，直接抛出拒绝执⾏的异常，即打断当前执⾏流程。

- 使用场景
    > 无特殊使用场景，使用自定义线程池实例时，正确处理抛出异常即可。
    > 注意该策略是`ThreadPoolExecutor`默认策略。但线程池使用无界队列，则不会触发拒绝策略。

- 代码

    ```java
    public static class AbortPolicy implements RejectedExecutionHandler {
        public AbortPolicy() {
        }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException (
                "Task " 
                + r.toString() 
                + " rejected from "
                + e.toString()
            );
        }
    }
    ```

### 丢弃策略（`DiscardPolicy`）

- 功能
    > 直接丢弃任务，不触发任何动作。

- 使⽤场景
    > 适用无关紧要的任务。

- 代码

```java
// 仅是一个空实现
public static class DiscardPolicy implements RejectedExecutionHandler {
    public DiscardPolicy() {
    }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}
```

### 弃⽼策略（`DiscardOldestPolicy`）

- 功能
    > 如果线程池未关闭，就弹出队列头部的元素，然后尝试执⾏。

- 使用场景
    > 会丢弃待执行优先级最高的任务。如，消息的发布、更新可使用。

- 代码

    ```java
    public static class DiscardOldestPolicy implements RejectedExecutionHandler {
        public DiscardOldestPolicy() {
        }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                e.getQueue().poll();
                e.execute(r);
            }
        }
    }
    ```

## 第三⽅实现的拒绝策略

### dubbo中的线程拒绝策略

- 功能
    1. 输出⼀条警告级别的⽇志。⽇志内容包括线程池的详细设置参数、线程池当前的状态，当前拒绝任务的⼀些详细信息。
    2. 输出当前线程堆栈详情。
    3. 继续抛出拒绝执⾏异常，使本次任务失败。这个继承了JDK默认拒绝策略的特性

- 代码

```java
public class AbortPolicyWithReport extends ThreadPoolExecutor.AbortPolicy { 
    protected static final Logger logger = LoggerFactory.getLogger(AbortPolicyWithReport.class); 
    private final String threadName; 
    private final URL url; 
    private static volatile long lastPrintTime = 0; 
    private static Semaphore guard = new Semaphore(1); 
    public AbortPolicyWithReport(String threadName, URL url) { 
        this.threadName = threadName; 
        this.url = url; 
    } 
    @Override 
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) { 
        String msg = String.format("Thread pool is EXHAUSTED!" 
            + " Thread Name: %s, Pool Size: %d (active: %d, core: %d, max: %d, largest: %d), Task: %d (completed: %d)," 
            + " Executor status:(isShutdown:%s, isTerminated:%s, isTerminating:%s), in %s://%s:%d!", 
            threadName, e.getPoolSize(), e.getActiveCount(), e.getCorePoolSize(), e.getMaximumPoolSize(), e.getLargestPoolSize(), 
            e.getTaskCount(), e.getCompletedTaskCount(), e.isShutdown(), e.isTerminated(), e.isTerminating(), url.getProtocol(), url.getIp(), url.getPort()); 
            
        logger.warn(msg); 
        dumpJStack(); 
        throw new RejectedExecutionException(msg); 
    } 
    private void dumpJStack() { 
       // ...
    } 
}
```

### Netty中的线程池拒绝策略

- 功能
    > 类似`CallerRunsPolicy`，线程池未关闭，则**新建**线程来处理。

- 代码

    ```java
    private static final class NewThreadRunsPolicy implements RejectedExecutionHandler {
        NewThreadRunsPolicy() {
            super();
        }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            try {
                final Thread t = new Thread(r, "Temporary task executor");
                t.start();
            } catch (Throwable e) {
                throw new RejectedExecutionException("Failed to start a new thread", e);
            }
        }
    }
    ```
