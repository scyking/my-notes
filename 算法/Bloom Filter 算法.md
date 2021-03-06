# Bloom Filter 算法

标签（空格分隔）： 算法

---

[toc]

## 介绍

> 用来查询某一数据是否在某一数据集合中。

### 优点

1. 节省空间：不需要存储数据本身，只需要存储数据对应hash比特位
1. 时间复杂度低：插入和查找的时间复杂度都为O(k)，k为哈希函数的个数

### 缺点

1. 存在假阳性：判断存在，可能出现元素不在集合中（判断准确率取决于哈希函数的个数）
1. 不能删除元素：如果一个元素被删除，但是却不能从布隆过滤器中删除（造成假阳性的原因之一）

### 使用场景

1. 爬虫系统url去重
1. 垃圾邮件过滤
1. 黑名单
1. 分布式缓存
1. 资源路由

## 算法描述

1. 首先需要k个hash函数，每个函数可以把key散列成为1个整数
2. 初始化时，需要一个长度为n比特的数组，每个比特位初始化为0
3. 某个key加入集合时，用k个hash函数计算出k个散列值，并把数组中对应的比特位置为1
4. 判断某个key是否在集合时，用k个hash函数计算出k个散列值，并查询数组中对应的比特位，如果所有的比特位都是1，认为在集合中。

## 基本思想

> 使用链表、树等数据结构存储元素集合，判断一个元素是否在集合中，会随着集合中元素增加，伴有存储空间需求增大、检索变慢的问题。
> 使用散列表（哈希表，Hash table）的数据结构。
> 通过一个Hash函数将一个元素映射成一个位阵列（`BitArray`）中的一个点。这样通过这个点是否为`1`，就可以判断集合中是否存在该元素。
> 为解决hash冲突的问题，使用多个Hash。

## 概率推导

> 假设 Hash 函数以等概率条件选择并设置 `Bit Array` 中的某一位，m 是该位数组的大小，k 是 Hash 函数的个数。

- 位数组中某一特定的位在进行元素插入时的 Hash 操作中没有被置位的概率是：`1 - 1/m`
- 在所有 k 次 Hash 操作后该位都没有被置 "1" 的概率是：`(1 - 1/m)^k`
- 插入了 n 个元素，那么某一位仍然为 "0" 的概率是：`(1 - 1/m)^kn`
- 插入了 n 个元素，那么某一位仍然为 "1" 的概率是：`1 - (1 - 1/m)^kn`
