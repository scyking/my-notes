# jvm 性能调优

标签（空格分隔）： java-jvm

---
[toc]

## 目标

在以下三点中，通过修改jvm参数寻找平衡。

- GC的时间足够的小
- GC的次数足够的少
- 发生Full GC的周期足够的长

## 方法

1. 减少使用全局变量和大对象；
2. 调整新生代的大小到最合适；
3. 设置老年代的大小为最合适；
4. 选择合适的GC收集器；

## 常用分析工具

- JConsole:可直接输入`jconsole`使用。
- VisualVM:可直接输入`jvisualvm`使用。
- `jps`:主要用来输出jvm中运行的进程状态信息。
- `jstack`:主要用来查看某个java进程内的线程堆栈信息。
- `jmap`:用来查看堆内存使用情况，一般结合jhat使用。
- `jstat`:jvm统计监测工具。

## 补充

常见的性能问题

1. cpu load 过高导致系统不可用或者tps(吞吐量)急剧降低
1. jvm 问题导致tps(吞吐量)降低
    - young gc 次数过多
    - full gc 次数过多
    - full gc 时间长
    - perm space 回收频繁或oom（Out Of Memory）
1. 重复查询泛滥
1. 锁竞争或死循环
1. 连接池连接占用
1. 不合理的使用集合




