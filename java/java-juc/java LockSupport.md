# java LockSupport

---

[toc]

## 概述

> 用来创建锁和其他同步类的基本线程阻塞原语。

### 核心方法

- `park()`，用来阻塞当前调用线程。
- `unpark()`，用于唤醒指定线程。

### 实现原理

> 使用 `Permit`（许可）实现阻塞和唤醒线程的功能。`Permit` 最多存在一个，线程阻塞会消耗 `Permit`。
> 可以把许可看成是一种 `(0,1)` 的 `Semaphore` （信号量），但与 `Semaphore` 不同的是，许可的累加上限是 `1`。
> 初始时，`permit`为`0`，当调用`unpark()`方法时，线程的`permit + 1`；当调用`park()`方法时，如果`permit`为`0`，则调用线程进入阻塞状态。

### 使用`wait`，`notify`阻塞唤醒线程缺点

1. `wait`和`notify`都是`Object`中的方法。在调用这两个方法前必须先获得锁对象，这限制了其使用场合:只能在同步代码块中。
1. 当对象的等待队列中有多个线程时，`notify`只能随机选择一个线程唤醒，无法唤醒指定的线程。

## 使用示例

```java
public class LockSupportTest {

    public static void main(String[] args) {
        Thread parkThread = new Thread(new ParkThread());
        parkThread.start();
        System.out.println("开始线程唤醒");
        LockSupport.unpark(parkThread);
        System.out.println("结束线程唤醒");

    }

    static class ParkThread implements Runnable{

        @Override
        public void run() {
            System.out.println("开始线程阻塞");
            LockSupport.park();
            System.out.println("结束线程阻塞");
        }
    }
}
```

## 注意事项

> 先唤醒线程，再阻塞线程，线程不会阻塞。（唤醒会获得凭证，因此不会阻塞）
> 先唤醒线程两次再阻塞两次时就会导致线程真的阻塞。（两次唤醒只会获得一个凭证，而两次阻塞会消耗两个凭证）
