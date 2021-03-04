# Collections.sort 用法

标签（空格分隔）： java

---

## 介绍

用于对`List`类型数据排序。

- 方法一

    ```
    public static <T extends Comparable<? super T>> void sort(List<T> list) {
        list.sort(null);
    }
    ```

    List中对象需实现`Comparable`接口并重写`compareTo`方法。

- 方法二

    ```
    public static <T> void sort(List<T> list, Comparator<? super T> c) {
        list.sort(c);
    }
    ```

    通过实现`Comparator`接口的`compare`方法来完成自定义排序。

## 实现举例

### 方法一

- `Integer`对象排序

    可直接调用，因为`Integer`对象已实现`Comparable`接口。

    ```
    List<Integer> intList = Arrays.asList(2, 3, 1, 5, 4, 6);
    Collections.sort(intList);
    ```

- 其它对象排序

    - `Person`对象
    
        实现`Comparable`接口，重写`compareTo`方法实现排序。
    
        ```
        public class Person implements Comparable<Person> {
        
        		private String name;
        		private int age;
        
        		public Person(String name, int age) {
        			this.setName(name);
        			this.setAge(age);
        		}
        
        		public String getName() {
        			return name;
        		}
        
        		public void setName(String name) {
        			this.name = name;
        		}
        
        		public int getAge() {
        			return age;
        		}
        
        		public void setAge(int age) {
        			this.age = age;
        		}
        
        		@Override
        		public int compareTo(Person o) {
        			// 按年龄正序排序
        			return this.getAge() - o.getAge();
        		}
        	}
        ```

    - 调用测试
    
        ```
        List<Person> list = new ArrayList<Person>();
		list.add(new Person("King", 11));
		list.add(new Person("Sky", 15));
		list.add(new Person("Kitty", 7));
		list.add(new Person("Yohn", 18));

		Collections.sort(list);
        ```
    
### 方法二

- `Person`对象

    对象无需实现`Comparable`接口。因此，方法二使用起来更加灵活。
    
    ```
    public class Person {

		private String name;
		private int age;

		public Person(String name, int age) {
			this.setName(name);
			this.setAge(age);
		}

		public String getName() {
			return name;
		}

		public void setName(String name) {
			this.name = name;
		}

		public int getAge() {
			return age;
		}

		public void setAge(int age) {
			this.age = age;
		}
	}
    ```
    
- 调用测试

    ```
    List<Person> list = new ArrayList<Person>();
	list.add(new Person("King", 11));
	list.add(new Person("Sky", 15));
	list.add(new Person("Kitty", 7));
	list.add(new Person("Yohn", 18));
		
    Collections.sort(list, new Comparator<Person>() {

		@Override
		public int compare(Person p1, Person p2) {
			// 按年龄正序排序
			return p1.getAge() - p2.getAge();
			// 按年龄倒序排序
			// return p2.getAge() - p1.getAge();
		}

	});
    ```

## 解析

通过查看源码，两种方法实现排序都使用了`Tim Sort`算法（结合了归并排序与插入排序）。调用`compareTo`或`compare`方法进行判断，影响最终的排序结果。所以，按需求重写`compareTo`或`compare`方法，可达成排序目标。

PS：版本为 `JDK 1.8` 。其它版本使用了不同的排序算法。






