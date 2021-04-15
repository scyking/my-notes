# idea 使用git

标签：idea

---

## 配置git环境

### 配置 Git
> 依次点击 File --> Setting --> Version Control --> Git，进行配置。
    
![Git 配置页面](https://images2018.cnblogs.com/blog/793754/201809/793754-20180913181112848-1206585689.png)
    
### 配置 GitHub
> 依次点击 File --> Setting --> Version Control --> GibHub，进行配置。
    
![Github 配置页面](https://img2018.cnblogs.com/blog/793754/201809/793754-20180913204843803-394725031.png)

### 添加 Repository

- 本地仓库创建
    - 创建项目
    - 依次点击 VCS --> Import into Version Control --> Create Git Repository
    - 选择本地仓库位置

- 文件颜色含义
    - 褐色：未add到git管理
    - 绿色：已add到git管理
    - 蓝色：文件有修改
    - 白色（普通色）：已提交文件

- 应用远程仓库
    1. 项目第一次 `push`,配置已有远程仓库地址
    ![配置页面](https://img2018.cnblogs.com/blog/793754/201809/793754-20180913221420622-1615921079.png)
    1. 将项目上传至远程仓库
    依次点击 VCS --> Import into Version Control --> Share Project on GitHub
    1. 从远程仓库克隆项目
    依次点击 File --> New --> Project from Version Control --> Git
        
## 回滚代码

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
    