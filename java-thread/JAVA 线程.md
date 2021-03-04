# JAVA 线程

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

## 实现方式
### 继承 Thread 类

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
### 实现 Runnable 接口

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

### 测试代码

```
public class Test {
    
    public static void main(String[] args){
        
        Thread ext1 = new ThreadExte("ext1");
        Thread ext2 = new ThreadExte("ext2");
        
        Runnable imp1 = new ThreadRunImpl("imp1");
        Thread t1 = new Thread(imp1);
        Runnable imp2 = new ThreadRunImpl("imp2");
        Thread t2 = new Thread(imp2);
        
        ext1.start();
        ext2.start();    
        
        t1.start();
        t2.start();
    }
}
```
### 实现Runnable的优势

1. 可以避免java单继承的限制。
1. 适合资源共享。



