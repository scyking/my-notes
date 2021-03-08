# java hashCode 与 equals

标签（空格分隔）： java

---

[toc]

## 介绍

### hashCode
> hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode() 定义在JDK的Object.java中，这就意味着Java中的任何类都包含有hashCode() 函数。

### equals
> equals它的作用也是判断两个对象是否相等，如果对象重写了equals()方法，比较两个对象的内容是否相等；如果没有重写，比较两个对象的地址是否相同，价于“==”。同样的，equals()定义在JDK的Object.java中，这就意味着Java中的任何类都包含有equals()函数。

## `hashCode()` 和 `equals()` 关系

### 不会创建“类对应的散列表”
> 即不会在`HashSet`, `HashTable`,`HashMap`等本质是散列表的数据结构中，用到该类。例如，不会创建该类的HashSet集合。
> 在这种情况下，该类的`hashCode()` 和 `equals()`没有关系。


### 会创建“类对应的散列表”
> 即会在HashSet, HashTable, HashMap等等这些本质是散列表的数据结构中，用到该类。例如，创建该类的HashSet集合。
> 在这种情况下，该类的`hashCode()` 和 `equals()`是有关系的。
> 因为在散列表中，键值对在散列表中的位置与`hashCode()`有关，即两个键值对是否冲突与`hashCode()`有关。

举个例子：
```
private static class Person {    
    private String name;
    private int age
    public Person(String name, int age) {      
        super();      
        this.name = name;      
        this.age = age;    
    }

    @Override    
    public boolean equals(Object obj) {     
        if (obj == null) {        
            return false;      
        }
        // 如果是同一个对象返回true，反之返回false      
        if (this == obj) {        
            return true;      
        }
        // 判断是否类型相同      
        if (this.getClass() != obj.getClass()) {        
            return false;      
        }      
        Person person = (Person) obj;      
        return name.equals(person.name) && age == person.age; 
    }
}
```

```
// 新建Person对象
Person p1 = new Person("eee", 100); 
Person p2 = new Person("eee", 100); 
Person p3 = new Person("aaa", 200);
// 新建HashSet对象 
HashSet<Person> set = new HashSet<>();
set.add(p1);
set.add(p2);
set.add(p3);
// 比较p1 和 p2， 并打印它们的hashCode()        
System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n", p1.equals(p2), p1.hashCode(), p2.hashCode());
// 打印set        
System.out.printf("set:%s\n", set); 
```

运行结果：
```
p1.equals(p2) : true; p1(2018699554) p2(1311053135)
set:[(eee, 100), (aaa, 200), (eee, 100)]
```
结果分析：

> 我们重写了Person的equals()。但是，HashSet中仍然有重复元素，这是因为虽然p1 和 p2的内容相等，但是它们的hashCode()不等；
> 所以，HashSet在添加p1和p2的时候，认为它们不相等。

解决办法，在`Person`类中添加`hashCode()`重写方法。
```
 @Override
 public int hashCode() {      
     int nameHash = name.toUpperCase().hashCode();      
     return nameHash ^ age;    
 }
```
 
### HashSet 使用 hashCode() 源码

hashSet使用的是hashMap的put方法：

```
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```
putVal方法：
```
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
 
hashMap的put方法，使用hashCode()用key作为参数计算出hash值，然后进行比较，如果相同，再通过equals()比较key值是否相同，如果相同，返回同一个对象。
所以，如果类使用在散列表的集合对象中，要判断两个对象是否相同，除了要覆盖equals()之外，也要覆盖hashCode()函数。否则，equals()无效。

### gc后Object的hashCode不发生改变
> Object对象第一次计算hashCode，会将结果存储在对象头中。即使内存地址发生改变，hashCode值也不会改变。
> 同一对象在执行期间若已经存储在集合中，则不能修改影响hashCode值的相关信息，否则会导致内存泄露问题。

## 重写hashCode的方法
> 一个好的hashCode的方法的目标：为不相等的对象产生不相等的散列码，同样的，相等的对象必须拥有相等的散列码。

1. 把某个非零的常数值，比如17，保存在一个int型的result中；
1. 对于每个关键域f（equals方法中设计到的每个域），作以下操作：
    - 为该域计算int类型的散列码；
        1. 如果该域是`boolean`类型，则计算(f?1:0),
        1. 如果该域是`byte`,`char`,`short`或者`int`类型,计算(int)f,
        1. 如果是`long`类型，计算(int)(f^(f>>>32)).
        1. 如果是`float`类型，计算Float.floatToIntBits(f).
        1. 如果是`double`类型，计算Double.doubleToLongBits(f),然后再计算long型的hash值
        1. 如果是对象引用，则递归的调用域的hashCode，如果是更复杂的比较，则需要为这个域计算一个范式，然后针对范式调用hashCode，如果为null，返回0
        1. 如果是一个数组，则把每一个元素当成一个单独的域来处理。
    - `result = 31 * result + c;`
1. 返回result
1. 编写单元测试验证有没有实现所有相等的实例都有相等的散列码。

举个例子：
```
@Override
public int hashCode() {  
    int result = 17;  
    result = 31 * result + name.hashCode();  
    return result;
}
```

### 使用`result = 31 * result + c;`原因
> 乘法使hash值依赖于域的顺序，如果没有乘法那么所有顺序不同的字符串String对象都会有一样的hash值，而31是一个奇素数，如果是偶数，并且乘法溢出的话，信息会丢失，
> 31有个很好的特性是`31*i ==(i<<5)-i`,即2的5次方减1，虚拟机会优化乘法操作为移位操作的。
> PS: 素数也称质数。奇素数是指不能被2整除而且因数只有1和它本身的正整数。




