# MQTT 最后遗嘱

---

[toc]

## 概述

> 最后遗嘱（LWT，Last Will and Testament），是由客户端预先定义好的主题和对应消息，附加在`CONNECT`的数据包中，包括遗愿主题、遗愿 QoS、遗愿消息等。

## 相关参数

- `Will Flag`：是否使用 LWT，1 开启
- `Will Topic`：遗愿主题名，不可使用通配符
- `Will Qos`：发布遗愿消息时使用的 QoS
- `Will Retain`：遗愿消息的 Retain 标识
- `Will Message`：遗愿消息内容

## 发送时机

> 当MQTT代理 `Broker` 检测到有客户端Client非正常断开连接时，再由服务器主动发布此消息，然后相关的订阅者会收到消息。

- `Broker` 检测到底层的 I/O 异常
- 客户端 未能在心跳 `Keep Alive` 的间隔内和 `Broker` 进行消息交互
- 客户端 在关闭底层 `TCP` 连接前没有发送 `DISCONNECT` 数据包
- 客户端 发送错误格式的数据包到 `Broker`，导致关闭和客户端的连接等

## 注意事项

> 当客户端通过发布 `DISCONNECT` 数据包断开连接时，属于正常断开连接，并不会触发 `LWT` 的机制，与此同时 `Broker` 还会丢弃掉当前客户端在连接时指定的相关 `LWT` 参数。
