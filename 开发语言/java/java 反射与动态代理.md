# java 反射与动态代理

标签（空格分隔）： java

---

[toc]

## 反射
> 反射的概念是由Smith在1982年首次提出的，主要是指程序可以访问、检测和修改它本身状态或行为的一种能力。
> Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法；这种动态获取信息以及动态调用对象方法的功能，称为java语言的反射机制。

### 反射的应用
1. Spring框架：IOC（控制反转）
2. Hibernate框架：关联映射等
3. 白盒测试

### 相关的API
1. java.lang包下
> Class<T> : 表示一个正在运行的java应用程序中的类和接口，是Reflection的起源。
2. java.lang.reflect包下
- Field类 : 代表类的成员变量，也称类的属性。
- Method类 : 代表类的方法。
- Constructor类 : 代表类的构造方法。
- Array类 : 提供了动态创建数组，以及访问数组元素的静态方法。

### 关于Class<T>
> 类是程序的一部分，每个类都有一个Class对象。换言之，每当编写并且编译了一个新类，就会产生一个Class对象。
> Class没有公共构造方法。Class对象是在加载类时由Java虚拟机以及通过调用类加载器中的defineClass方法自动构造的，因此不能显式地声明一个Class对象。
> Class是Reflection的起源。要想操纵类中的属性和方法，都必须从获取Class  Object开始，获取方式如下表。

### 获取Class方式

|获取方式|说明|示例
|---
|`Object.getClass()`|获取指定实例对象的Class|`List  li  =  new ArrayList();`<br>`Class liCla  =  li.getClass();`
|`Class.getSuperclass()`|获取当前Class的继承类Class|`List  li  =  new ArrayList();`<br>`Class liCla  =  li.getClass();`<br>`Class suCla =  liCla.getSuperclass();`
|`Object.class`|.class直接获取|`Class liCla = ArrayList.class;`
|`Class.forName("类名")`|用class的静态方法，传入类的全称即可|`Class cla = Class.forName("java.util.ArrayList");`
|`Primitive.TYPE`|基本数据类型的封装类获取Class的方式|`Class longClass = Long.TYPE;`<br> `Class inteClass = Integer.TYPE;`

## 代理模式

```
    //  定义接口
    interface Subject{
        void sayHi();
        void sayHello();
    }
    
    // 定义被代理对象：具有自己的业务实现
    static class SubjectImpl implements Subject{
        @Override
        public void sayHi() {
            System.out.println("hi");
        }
    }
```

### 静态代理

- 事先写好代理类，可以手工编写，也可以用工具生成。
- 缺点是每个业务类都要对应一个代理类，非常不灵活。

```
    // 定义代理对象：对被代理对象封装，添加自己的业务逻辑
    static class SubjectImplProxy implements Subject{
        private Subject target;
        
        public SubjectImplProxy(Subject target) {
            this.target = target;
        }
        
        @Override
        public void sayHi() {
            System.out.print("say:");
            target.sayHi();
        }
    }

    public static void main(String[] args) {
        // 1. 声明被代理对象
        Subject subject = new SubjectImpl();
        // 2. 封装被代理对象，对其进行代理
        Subject subjectProxy = new SubjectImplProxy(subject);
        // 3. 测试输出结果
        subjectProxy.sayHi();
    }
```

### 动态代理

- 运行时自动生成代理对象。在某些情况下，一个对象不适合或者不能直接引用另一个对象，而代理对象可以在两者之间起到中介的作用（可类比房屋中介，房东委托中介销售房屋、签订合同等）。
- 实现阶段不用关心代理谁，而是在运行阶段才指定代理哪个一个对象（不确定性）。
- 缺点是生成代理代理对象和调用代理方法都要额外花费时间。

```
    // 与静态代理相比，此处不需实现接口(Subject)
    static class ProxyInvocationHandler implements InvocationHandler {
        private Subject target;

        public ProxyInvocationHandler(Subject target) {
            this.target = target;
        }

       // 此处对被代理对象统一添加相同业务处理，解决静态代理相同业务代码需重复书写的问题。
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.print("say:");
            // 当代理对象调用被代理对象的方法时，其会自动的跳转到代理对象关联的handler对象的invoke方法来进行调用
            return method.invoke(target, args);
        }
    }

    public static void main(String[] args) {
        // 1. 创建被代理对象
        Subject subject = new SubjectImpl();
        // 2. 通过 Proxy 动态创建代理对象
        Subject subjectProxy = (Subject) Proxy.newProxyInstance(subject.getClass().getClassLoader(), subject.getClass().getInterfaces(), new ProxyInvocationHandler(subject));
        // 3. 测试输出结果
        subjectProxy.sayHi();

    }
```

#### jdk动态代理
> 面向接口
> 基于Java反射机制实现，必须要实现了接口的业务类才能用这种办法生成代理对象。
> 采用的JDK提供的动态代理接口InvocationHandler来实现动态代理类。其中invoke方法是该接口定义必须实现的，它完成对真实方法的调用。

1. 优点
- 最小化依赖关系，减少依赖意味着简化开发和维护，JDK 本身的支持，可能比 cglib 更加可靠。
- 平滑进行 JDK 版本升级，而字节码类库通常需要进行更新以保证在新版 Java 上能够使用。
- 代码实现简单。

#### cglib动态代理
> 通过字节码底层继承要代理类来实现，因此如果被代理类被final关键字所修饰，会失败。
> 利用ASM开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

1. ASM
> ASM 是一个 Java 字节码操控框架。它能够以二进制形式修改已有类或者动态生成类。ASM 可以直接产生二进制 class 文件，也可以在类被加载入 Java 虚拟机之前动态改变类行为。ASM 从类文件中读入信息后，能够改变类行为，分析类信息，甚至能够根据用户要求生成新类。

1. 优点
- 有的时候调用目标可能不便实现额外接口，从某种角度看，限定调用者实现接口是有些侵入性的实践，类似 cglib 动态代理就没有这种限制。
- 只操作我们关心的类，而不必为其他相关类增加工作量。
- 高性能，速度更快。

1. 示例

    ```
    static public class SubjectImplCglib implements MethodInterceptor {
        private SubjectImpl target;

        public Object getInstance(SubjectImpl target) {
            this.target = target;
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(this.target.getClass());
            // 设置回调方法
            enhancer.setCallback(this);
            // 创建代理对象
            return enhancer.create();
        }

        @Override
        public Object intercept(Object object, Method method, Object[] args, MethodProxy proxy) throws Throwable {
            System.out.print("say:");
            return proxy.invokeSuper(object, args);
        }
    }
    
    public static void main(String[] args){
        // 通过cglib动态代理对象
        SubjectImplCglib cglib = new SubjectImplCglib();
        SubjectImpl subjectImpl = (SubjectImpl) cglib.getInstance(new SubjectImpl());
        // 测试输出结果
        subjectImpl.sayHi();
    }
    ```









