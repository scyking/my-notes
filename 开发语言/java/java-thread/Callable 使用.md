# Callable 使用

标签（空格分隔）： java-thread

---

## 问题

继承 `Thread` 类或者实现 `Runnable` 接口，`run()`方法返回类型是`void`，即启动的线程任务无返回结果。

## 解决

1. 实现 `Callable` 接口

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

2. 执行线程任务
```
		ExecutorService exec = Executors.newCachedThreadPool();
		ArrayList<Future<String>> results = new ArrayList<Future<String>>();
		for (int i = 0; i < 10; i++) {
			// ExecutorService.submit()产生Future对象
			results.add(exec.submit(new TaskWithResult(i)));
		}
```
3. Future对象
	- `isDone()`，判断任务是否执行完成。
	- `get()`，获取任务执行结果，任务未完成则阻塞。




