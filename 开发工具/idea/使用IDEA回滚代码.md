# 使用IDEA回滚代码

标签（空格分隔）：bug git idea

---

[TOC]

## 需求

因为某种原因，需要将本地代码及远程代码回滚至某一版本。

## 解决方法

1. 获取当前代码版本号`new version`及旧代码版本号`old version`（要回滚至的版本）
    - 右键项目，点击`Git`
    - 点击`Show History`
    - 点击相应版本，右键`Copy Revision Number`

1. 将本地代码回滚至旧代码版本
    - 右键项目，点击`Git`
    - 点击`Repository`
    - 点击`Rest Head`
    - `Reset Type`选择`Hard`，`To Commit`输入旧代码版本号`old version`，点击`Reset`

1. 回滚提交状态后可重新提交
    - 右键项目，点击`Git`
    - 点击`Repository`
    - 点击`Rest Head`
    - `Reset Type`选择`Mixed`，`To Commit`输入当前代码版本号`new version`，点击`Reset`
    - 代码可重新提交

## 注意事项

1. 未提交的修改项会全部丢失
    


