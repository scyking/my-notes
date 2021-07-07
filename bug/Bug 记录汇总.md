# Bug 记录

标签（空格分隔）： bug

[toc]

---

## MongoDB关联查询失效
原因：`ObjectId` 不能同 `String` 进行匹配，字段类型不同，匹配会失败。

## 拦截器注入`service`为空
原因：拦截器要以`bean`的形式注入，不能直接`new`进行配置。

## 跨域导致拦截器获取不到自定义`header`信息
解决：修改跨域配置，使用`Filter`。`Filter`过滤器会在拦截器之前生效。

## 通过`Long.toHexString()`生成的字符串，调用`Long.parseLong()`转`long`失败
原因：`Long.toHexString()`是无符号转换，`Long.parseLong()`是有符号转换。数值过大会存在转换问题

## 导出文件报错`HttpMessageNotWritableException`
报错部分信息：
```
org.springframework.http.converter.HttpMessageNotWritableException: No converter for [class java.lang.Boolean] with preset Content-Type 'application/octet-stream;charset=UTF-8'
```
解决：
将`controller`中的导出方法返回类型改为`void`
原因：
多次response导致，http server发送response后就关闭了socket，这个时候再次发送response给http client就会出现这个问题。

## docker部署微服务，导出文件乱码
问题：

1. 使用`docker`部署`springboot`服务
1. `encoding`已经统一设置为`UTF-8`
1. 本地测试无问题
1. linux服务器执行`locale`命令，配置也均为`UTF-8`
1. 进入`docker`容器，执行`locale`命令，编码配置非`UTF-8`

解决：
在`dockerfile`中添加环境变量
```
ENV LANG C.UTF-8
ENV LANGUAGE zh_CN.UTF-8
ENV LC_ALL C.UTF-8
```

## gateway路由带下划线服务名失效
原因：构造URI时，字符串如果有下划线无法解析host。
解决：gateway配置uri不能包含下划线，所以要去掉服务名中的下划线。

## 使用docker部署微服务，gateway路由多节点服务，找不到服务实例。
原因：`eureka.instance.prefer-ip-address`设置为`true`
解决：`eureka.instance`中添加如下配置
```
prefer-ip-address: false
instance-id: ${eureka.instance.ip-address}:${spring.application.name}:${server.port}
hostname: ${eureka.instance.ip-address}
ip-address: ${SERVER_HOST}
```

## mongoDB 时间范围查询报`No property XXX found for type ...`
原因：属性名为`XXX_XX`,并使用`.and`构造查询条件，导致`_XX`部分解析失败。具体原因待分析JPA源码。
解决：使用`Criteria.where`构造查询条件。

## java下载Zip压缩文件，解压后报不可预料的压缩文件末端
原因：ZipOutputStream的流没有关闭
解决：返回respones之前手动关闭ZipOutputStream的流

## 下载excel文件，打开出错
原因：使用`byte[] bytes = excel.getBytes()`获取到的文件流不完整
解决：使用下面方式获取
```
ByteArrayOutputStream os = new ByteArrayOutputStream();
excel.write(os);
```

## 下载文件，文件名乱码
问题描述：浏览器`response`中`filename`乱码。直接调用端口下载没有问题，但前端使用`filename`命名下载文件，就会出现问题。
解决：使用`MimeUtility.encodeText(filename, "UTF8", "B");`进行base64编码，前端再解析。直接使用`MimeUtility`，maven打包会出现问题，所以手动拼接`=?UTF8?B?`前缀，`?=`后缀。

## nginx 一个`server`配置多个`location`，访问报 `404`
原因：`root` 是指定目录的上级目录
解决：使用 `alias` 

## linux上验证码不显示
错误信息：
``` 
ERROR [io.undertow.request] (default task-118) UT005023: Exception handling request to /captchacode: java.lang.NoClassDefFoundError: Could not initialize class sun.awt.X11GraphicsEnvironment
```
解决办法：
```
// 在验证码相关类上添加如下代码
static{
 	System.setProperty("java.awt.headless", "true");
 }
```

## (JDK7)三目运算中自动装箱、拆箱引起NPE
源码：
```
Map<String, Boolean> map = new HashMap<String, Boolean>();
Boolean b = (map != null ? map.get("test") : false);
```
原因：(JDK7)三目运算符第二、三操作数类型不同（一个为对象，一个为基本数据类型），会将对象自动拆箱为基本数据类型。
拆箱过程：
```
Map<String, Boolean> map = new HashMap<String, Boolean>();
Boolean b = Boolean.valueOf(map ！= null ? map.get("test").booleanValue() : false);
```
解决：
```
Map<String, Boolean> map = new HashMap<String, Boolean>();
Boolean b = (map != null ? map.get("test") : Boolean.FALSE);
```
PS:(JDK8)中处理方式不同，不会引起NPE。

## linux执行sh报错：$’\r’: 未找到命令的解决
> 其他正常脚本，执行报错，也可能是格式问题。

原因：sh文件直接从windows复制过来导致的。

解决：

```
# 安装 dos2unix
yum install dos2unix
# 进行格式转换
dos2unix **.sh 
```

## linux没有那个文件或目录

问题描述：
文件存在，却提示没有那个文件或目录。

原因：
文件类型是dos。

解决：
使用dos2unix工具进行转换。

## apt-get update 过期

错误信息：
```
E: Release file for http://ftp.at.debian.org/debian-backports/dists/squeeze-backports/Release is expired (invalid since 1d 4h 40min 18s). Updates for this repository will not be applied.
```

错误原因：
系统时间与实际不一致。

解决办法：
同步本地时间。

## VirtualBox 中 centos7 下 ping 命令出现 Network is unreachable 问题的解决方法

错误原因：
IP 地址和网关配置不正确。查看命令如下：

```
# 查看 enp0s3 的有线网络配置
nmcli connection show enp0s3
```

解决办法：

1. 网络模式设置为桥接网络
2. 执行如下命令：

```
# 设置 centos7 虚拟机的网络获取方式为自动获取 IP
nmcli connection modify enp0s3 \
connection.autoconnect yes \
ipv4.method auto
# 生效配置
nmcli connection up enp0s3
```