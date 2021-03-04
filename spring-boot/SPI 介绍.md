# SPI 介绍

标签（空格分隔）： spring-boot java

---

[toc]

## SPI（Service Provider Interface）机制

1. 思想
为某个接口寻找服务实现。

1. 约定
当服务的提供者，提供了服务接口的一种实现之后，在`jar`包的`META-INF/services/`目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该`jar`包`META-INF/services/`里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。通过这个约定，就不需要把服务放在代码中了，通过模块被装配的时候就可以发现服务类了。

## java 类加载机制
> java中的类加载器负载加载来自文件系统、网络或者其他来源的类文件。jvm的类加载器默认使用的是双亲委派模式。

1. 类加载器
- `BootstrapClassLoader`（启动类加载器）
`c++`编写，加载`java`核心库`java.*`,构造`ExtClassLoader`和`AppClassLoader`。负责加载JDK自带的`rt.jar`包中的类文件，是所有类加载的父类。

- `ExtClassLoader` （标准扩展类加载器）
`java`编写，加载扩展库，如`classpath`中的`jre/lib/ect` ，`javax.*`或者
`java.ext.dir` 指定位置中的类，开发者可以直接使用标准扩展类加载器。

- `AppClassLoader`（系统类加载器）
`java`编写，加载程序所在的目录，如`user.dir`所在的位置的`class`

- `CustomClassLoader`（用户自定义类加载器）
`java`编写,用户自定义的类加载器,可加载指定路径的`class`文件

1. 双亲委派模式
- 原理
当一个类加载器收到类加载任务时，会先交给自己的父加载器去完成，因此最终加载任务都会传递到最顶层的BootstrapClassLoader，只有当父加载器无法完成加载任务时，才会尝试自己来加载。

- 源码
    ```
    protected Class<?> loadClass(String name, boolean resolve) {
        synchronized (getClassLoadingLock(name)) {
        // 首先，检查该类是否已经被加载，如果从JVM缓存中找到该类，则直接返回
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            try {
                // 遵循双亲委派的模型，首先会通过递归从父加载器开始找，
                // 直到父类加载器是BootstrapClassLoader为止
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {}
            if (c == null) {
                // 如果还找不到，尝试通过findClass方法去寻找
                // findClass是留给开发者自己实现的，也就是说
                // 自定义类加载器时，重写此方法即可
               c = findClass(name);
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
        }
    }
    ```
- 缺陷
在双亲委派模型中，子类加载器可以使用父类加载器已经加载的类，而父类加载器无法使用子类加载器已经加载的。这就导致了双亲委派模型并不能解决所有的类加载器问题。

1. 使用线程上下文类加载器(ContextClassLoader)加载
- 如果不做任何的设置，Java应用的线程的上下文类加载器默认就是AppClassLoader。在核心类库使用SPI接口时，传递的类加载器使用线程上下文类加载器，就可以成功的加载到SPI实现的类。线程上下文类加载器在很多SPI的实现中都会用到。
- 通常我们可以通过`Thread.currentThread().getClassLoader()`和`Thread.currentThread().getContextClassLoader()`获取线程上下文类加载器。

1. 使用类加载器加载资源文件，比如jar包

- 类加载器除了加载class外，还有一个非常重要功能，就是加载资源，它可以从jar包中读取任何资源文件。比如，`ClassLoader.getResources(String name)`方法就是用于读取jar包中的资源文件

    ```
    //获取资源的方法
    public Enumeration<URL> getResources(String name) throws IOException {
        Enumeration<URL>[] tmp = (Enumeration<URL>[]) new Enumeration<?>[2];
        if (parent != null) {
            tmp[0] = parent.getResources(name);
        } else {
            tmp[0] = getBootstrapResources(name);
        }
        tmp[1] = findResources(name);
        return new CompoundEnumeration<>(tmp);
    }
    ```

## springboot中的类SPI扩展机制

1. 介绍
在`springboot`的自动装配过程中，会加载`META-INF/spring.factories`文件，而加载的过程是由`SpringFactoriesLoader`加载的。

1. 源码

    ```
    public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
    // spring.factories文件的格式为：key=value1,value2,value3
    // 从所有的jar包中找到META-INF/spring.factories文件
    // 然后从文件中解析出key=factoryClass类名称的所有value值
    public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
        String factoryClassName = factoryClass.getName();
        // 取得资源文件的URL
        Enumeration<URL> urls = (classLoader != null ? classLoader.getResources(FACTORIES_RESOURCE_LOCATION) : ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
        List<String> result = new ArrayList<String>();
        // 遍历所有的URL
        while (urls.hasMoreElements()) {
            URL url = urls.nextElement();
            // 根据资源文件URL解析properties文件，得到对应的一组@Configuration类
            Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
            String factoryClassNames = properties.getProperty(factoryClassName);
            // 组装数据，并返回
            result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
        }
        return result;
    }
    ```



