# java 赋值的别名现象

标签（空格分隔）： java

---

[toc]

## 赋值
> 基本类型存储了实际的数值，而并非指向一个对象的引用，所以在为其赋值的时候，是直接将一个地方的内容复制到了另一个地方。
> 一般对象赋值是操作引用，即直接操作引用的内容。

## 别名现象

```
@Data
class  People{
    int age;
    
    People(int age){
            this.age = age;
        }
}
```

```
People people1 = new People(17);
People people2 = new People(57);

// output:"people1 age:17;people2 age:57"
System.out.println("people1 age:" + people1.getAge() + ";people2 age:" + people2.getAge());
// 将people1的引用指向people2对象。即people1、people2引用相同对象，而原people1引用对象不再被引用，会被gc回收。
people1 = people2;
// output:"people1 age:57;people2 age:57"
System.out.println("people1 age:" + people1.getAge() + ";people2 age:" + people2.getAge());
// 此时people1引用已指向people2对象，修改现people1引用对象的值时，people2引用对象得相应值也会变化。（别名现象）
people1.setAge(25);
// output:people1 age:25;people2 age:25
System.out.println("people1 age:" + people1.getAge() + ";people2 age:" + people2.getAge());
```

## 不变类

```
// 不变类
String str1 = "str1";
String str2 = "str2";
str1 = str2;
// output:"str1:str2;str2:str2"
System.out.println("str1:" + str1 + ";str2:" + str2);
str1 = "test";
// output:"str1:test;str2:str2"
System.out.println("str1:" + str1 + ";str2:" + str2);
```




