# OpenSSL 生成证书

---

[toc]

## 概述

### OpenSSL

> OpenSSL 是一个开放源代码的软件库包，应用程序可以使用这个包来进行安全通信，可以避免信息被窃听到。

### SSL

> 安全套接层协议（Secure Sockets Layer），保证两个应用间通信的保密性和可靠性，可在服务器端和用户端同时实现支持。

### 非对称加密

> 用密钥对数据进行加密，然后使用另一个不同的密钥对数据进行解密。

### 数字签名

> 一种类似写在纸上的普通物理签名，使用公钥加密领域的技术实现，用于鉴别数字信息的方法。
> 通过 **Hash算法** 算出报文摘要，如果两个摘要一致，说明数据未被篡改，证明数据的完整性。

### 数字证书

> 数字证书是互联网通讯中标志通讯各方身份信息一串数字。
> 使用 `CA` 的私钥将网站的公钥等信息进行了签名，当客户端请求服务器端的时候，网站会把证书发给客户端，客户端先通过 `CA` 的数字签名校验 1 的身份，来证明证书的真实完整性。

## 生成自签证书

### 生成 `RSA` 私钥

```bash
## 生成 server.key
openssl genrsa -des3 -out server.key 2048
Enter pass phrase for server.key:                   // 填写密码
Verifying - Enter pass phrase for server.key:       // 确认密码

## 查看 server.key 加密内容
openssl rsa -text -in server.key
## 查看 server.key
cat server.key
```

- `-des3`，指定算法，为des3
- `2048`，指定算法强度，为2048位
- `-out`，指定秘钥输出文件

### 生成 `CSR` 证书签名请求文件

```bash
## 生成 server.csr
openssl req -new -key server.key -out server.csr
Country Name (2 letter code) []:CN                          // 输入国家代码，中国填写 CN
State or Province Name (full name) []:                      // 输入省份
Locality Name (eg, city) []:                                // 输入城市
Organization Name (eg, company) []:                         // 输入组织机构
Organizational Unit Name (eg, section) []:                  // 输入机构部门
Common Name (eg, fully qualified host name) []:*.abc.com    // 输入域名
Email Address []:                                           // 邮箱地址
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:                                    // 证书密码，如果不想设置密码，可以直接回车

## 查看 server.csr
openssl req -text -in server.csr -noout
```

- `-key`：指定ca私钥
- `-out`：指定csr输出文件

### 生成 `CA` 证书

```bash
## 生成 server.crt 文件。包含证书持有人的信息、公钥、以及签署者的签名等信息
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

- `x509`: 指定格式
- `-in`: 指定请求文件
- `-signkey`: 自签名

### 生成客户端证书

> 生成方式与上面类似。

```bash
## 生成私钥，client.key
openssl genrsa -out client.key 2048

## 生成请求文件，client.csr
openssl req -new -key client.key -out client.csr
Country Name (2 letter code) []:CN                          // 输入国家代码，中国填写 CN
State or Province Name (full name) []:                      // 输入省份
Locality Name (eg, city) []:                                // 输入城市
Organization Name (eg, company) []:                         // 输入组织机构
Organizational Unit Name (eg, section) []:                  // 输入机构部门
Common Name (eg, fully qualified host name) []:*.abc.com    // 输入域名
Email Address []:                                           // 邮箱地址
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:                                    // 证书密码，如果不想设置密码，可以直接回车

## 发给ca签名，client.crt文件
openssl x509 -req -days 365 -in client.csr -signkey client.key -out client.crt
```

## 生成证书脚本

> 编写 `Makefile` ，执行 `make` 命令生成证书。

```Makefile
CRT_LOCATION = certs                # 生成证书存放路径
CN = CN                             # 输入国家代码，中国填写 CN
O = abc                             # 输入组织机构
OU = abc                            # 输入机构部门
EA = *.abc.com                      # 邮箱地址
CRT_FILE_NAME = client              # 客户端证书名称

all: clean_certs ca server_crt

# CA name and key is "ca".
ca:
 openssl req -newkey rsa:2048 -x509 -nodes -sha512 -days 1095 \
    -keyout $(CRT_LOCATION)/ca.key -out $(CRT_LOCATION)/ca.crt -subj "/CN=localhost/O=abc/OU=abc/emailAddress=*.abc.com"

server_cert:
 # Create server key and CSR.
 openssl req -new -sha256 -newkey rsa:4096 -nodes -keyout $(CRT_LOCATION)/server.key \
    -out $(CRT_LOCATION)/server.csr -subj "/CN=$(CN)/O=$(O)/OU=$(OU)/emailAddress=$(EA)"

 # Sign server CSR.
 openssl x509 -req -days 1000 -in $(CRT_LOCATION)/server.csr -CA $(CRT_LOCATION)/ca.crt -CAkey $(CRT_LOCATION)/ca.key -CAcreateserial -out $(CRT_LOCATION)/server.crt
 
 # Remove CSR.
 rm $(CRT_LOCATION)/server.csr

client_cert:
 # Create client key and CSR.
 openssl req -new -sha256 -newkey rsa:4096 -nodes -keyout $(CRT_LOCATION)/$(CRT_FILE_NAME).key \
    -out $(CRT_LOCATION)/$(CRT_FILE_NAME).csr -subj "/CN=$(CN)/O=$(O)/OU=$(OU)/emailAddress=$(EA)"

 # Sign client CSR.
 openssl x509 -req -days 730 -in $(CRT_LOCATION)/$(CRT_FILE_NAME).csr -CA $(CRT_LOCATION)/ca.crt -CAkey $(CRT_LOCATION)/ca.key -CAcreateserial -out $(CRT_LOCATION)/$(CRT_FILE_NAME).crt
 
 # Remove CSR.
 rm $(CRT_LOCATION)/$(CRT_FILE_NAME).csr

clean_certs:
 rm -r $(CRT_LOCATION)/*.crt
 rm -r $(CRT_LOCATION)/*.key
```
