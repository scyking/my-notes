# java Map 原理分析

标签（空格分隔）： java

---

## HashMap

### 数据结构

数据结构是“链表散列”。即底层是一个数组结构，数组中的每一项又是一个链表。

### hash冲突解决

- 使用链表存储相同散列值的键值对。
- `hash()`计算方法，添加高位计算（扰动函数），防止低位不变，高位变化时，造成的 hash 冲突。

### 速度上的优化

- 底层数组长度使用2的n次方。
- 添加扰动函数，使散列更均匀。

### HashMap 扩容

1. 什么时候扩容？

    当 HashMap 中的元素个数超过`数组长度 * loadFactor（负载因子，默认值0.75）`。

1. 优化

    扩容会重新计算HashMap中元素在数组中的位置，这是一个消耗性能的操作。结合使用场景，通过`HashMap(int initialCapacity, float loadFactor)`指定数组初始化长度及负载因子值，减少扩容以提高使用性能。

### 遍历方式

```
Map map = new HashMap();
Iterator iter = map.entrySet().iterator();
while (iter.hasNext()) {
    // 不直接使用 “Object key = iter.next();” ，以提高效率。
    Map.Entry entry = (Map.Entry) iter.next();
    Object key = entry.getKey();
    Object val = entry.getValue();
}
```

### Fail-Fast 机制

一种错误检测机制。当多个线程对同一个集合的内容进行操作时，就可能会产生Fail-Fast事件。例如，在使用迭代器的过程中，有其他线程修改了map，将抛出 `ConcurrentModificationException`。

## HashTable

`HashTable` 已被淘汰。不需要线程安全，使用 `HashMap`。需要线程安全，使用 `ConcurrentHashMap`。

### 与HashMap比较

- Hashtable 不允许 key 和 value 为 null，而 HashMap 允许。
- Hashtable 方法是同步，而 HashMap 则不是。

## LinkedHashMap

LinkedHashMap 是 HashMap 的一个子类，保留了元素插入的顺序。

### 数据结构

与 HashMap 相比，LinkedHashMap 维护一个双向链表，用于存储迭代顺序。

### 排序模式

通过使用 `LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder)` 中`accessOrder`参数指定排序模式。

- `accessOrder = true` 访问顺序

- `accessOrder = false` 插入顺序（默认）

## ConcurrentHashMap

ConcurrentHashMap 是线程安全的。

### 数据结构

在 HashMap 的基础上，添加分段锁，保证线程安全。

### JDK8 实现

底层依然由“数组+链表+红黑树”的方式实现。在线程安全方面，摒弃了分段锁思想，利用[CAS](https://www.cnblogs.com/wscy/p/9415245.html)算法重新实现。

