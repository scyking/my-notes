# java 线程介绍

标签（空格分隔）： java-thread

---
[toc]


## 概念

### 同步与异步
> 关注的是消息通信机制 (synchronous communication/ asynchronous communication)

- 同步
就是在发出一个*调用*时，在没有得到结果之前，该*调用*就不返回。但是一旦调用返回，就得到返回值了。换句话说，就是由*调用者*主动等待这个*调用*的结果。
- 异步
与同步相反，*调用*在发出之后，这个调用就直接返回了，所以没有返回结果。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在*调用*发出后，*被调用者*通过状态、通知来通知调用者，或通过回调函数处理这个调用。

### 阻塞与非阻塞
> 关注的是程序在等待调用结果（消息，返回值）时的状态

- 阻塞
指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
- 非阻塞
指在不能立刻得到结果之前，该调用不会阻塞当前线程。

## 线程生命周期

### 线程状态转换

<img src="/md/java-thread/202141515482081.png" width="50%" alt="线程状态转换">

### 线程状态

1. 新建（new）

	创建线程。`Thread t = new MyThread();`

2. 就绪（runnable）

	线程处于可随时被cpu调度状态。执行`start()`启动线程。
	
3. 运行（running）

	线程被cpu调度，持续运行。调用`yield()`可让出cpu资源（不一定生效）。

4. 阻塞（blocked）

	处于`running`状态中的线程由于某种原因，暂时放弃对CPU的使用权，停止执行，直到其进入到就绪状态，才有机会再次被CPU调度。

	- 等待阻塞：执行`wait()`使线程挂起（会释放锁）。直到线程得到`notify()`或`notifyAll()`消息，线程才会进入`runnable`状态。

	- 同步阻塞：获取`synchronized`同步锁失败。

	- 其他阻塞：

        - 执行`sleep(milliseconds)`（中止执行给定时间，不会释放锁）；
        - 执行`join()`加入一个线程，原线程被挂起，直到目标线程结束；
        - 发出了`I/O`请求。

5. 死亡（dead）

	线程运行结束（线程任务执行完成/异常退出）。

## 实现方式
### 继承 `Thread` 类

```
public class ThreadExte extends Thread{
    
    private String name;

    public ThreadExte() {

    }

    public ThreadExte(String name) {

        this.name = name;
    }

    @Override
    public void run() {

        for (int i = 0; i < 5; i++) {

            System.out.println(name + ":" + i);
        }
    }

}
```
### 实现 `Runnable` 接口

```
public class ThreadRunImpl implements Runnable {

    private String name;

    public ThreadRunImpl() {

    }

    public ThreadRunImpl(String name) {

        this.name = name;
    }

    @Override
    public void run() {

        for (int i = 0; i < 5; i++) {

            System.out.println(name + ":" + i);
        }
    }
}
```

### 实现 `Callable` 接口
> 解决继承 `Thread` 类或者实现 `Runnable` 接口启动的线程任务无返回结果问题。

- 创建任务

```
public class TaskWithResult implements Callable<String> {

	private int id;

	public TaskWithResult(int id) {
		this.id = id;
	}

	public String call() {
		return "result of TaskWithResult " + id;
	}
}
```

ps:`Callable<String>`中`String`是线程任务返回结果的数据类型，与`call()`方法返回类型对应。

- 执行线程任务

```
ExecutorService exec = Executors.newCachedThreadPool();
ArrayList<Future<String>> results = new ArrayList<Future<String>>();
for (int i = 0; i < 10; i++) {
	// ExecutorService.submit()产生Future对象
	results.add(exec.submit(new TaskWithResult(i)));
}
```

- Future对象
    1. `isDone()`，判断任务是否执行完成。
	1. `get()`，获取任务执行结果，任务未完成则阻塞。