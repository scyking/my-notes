# linux 内核介绍

标签（空格分隔）： linux

---

[toc]

## 概述

> 内核是操作系统的核心。

### 主要功能

1. 响应中断，执行中断服务程序
1. 管理多个进程，调度和分享处理器的时间
1. 管理进程地址空间的内存管理
1. 网络和进程间通信等系统服务程序

### 活动范围

1. 运行于用户空间，执行用户进程
1. 运行于内核空间，处于进程上下文，代表某个特定进程的执行
1. 运行于内核空间，处于中断上下文，与任何进程无关，处理某个特定的中断

## Linux 版本号

> Linux 的版本号分为两部分，即内核版本与发行版本。

### 内核版本号组成

1. 形式如`A.B.C`。含义如下：

- `A`，内核主版本号。这是很少发生变化，只有当发生重大变化的代码和内核发生才会发生。
- `B`，内核次版本号。是指一些重大修改的内核。**偶数**表示稳定版本；**奇数**表示开发中版本。
- `C`，内核修订版本号。是指轻微修订的内核。这个数字当有安全补丁,bug修复，新的功能或驱动程序，内核便会有变化。

1. 形式如`major.minor.patch-build.desc`，含义如下：

- `major` : 主版本号，有结构变化才变更
- `minor` : 次版本号，新增功能时才发生变化，一般`奇数`表示测试版，`偶数`表示生产版
- `patch` : 补丁包数或次版本的修改次数
- `build` : 编译（或构建）的次数，每次编译可能对少量程序做优化或修改，但一般没有大的（可控的）功能变化。
- `desc`  : 当前版本的特殊信息，其信息由编译时指定，具有较大的随意性。如下常用标识：
  - `rc`（或`r`），表示发行候选版本（`release candidate`），rc后的数字表示该正式版本的第几个候选版本，多数情况下，各候选版本之间数字越大越接近正式版。
  - `smp`，表示对称多处理器（`Symmetric MultiProcessing`）。
  - `pp`，在Red Hat Linux中常用来表示测试版本（`pre-patch`）。
  - `EL`，在Red Hat Linux中用来表示企业版Linux（`Enterprise Linux`）。
  - `mm`，表示专门用来测试新的技术或新功能的版本。
  - `fc`，在Red Hat Linux中表示`Fedora Core`。  

### CentOS 下示例

> 用命令`uname -a`查看内核版本号。

```bash
Linux localhost.localdomain 3.10.0-1160.31.1.el7.x86_64 #1 SMP Thu Jun 10 13:32:12 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
```

说明如下：

- `3.10.0`
  - `3`, 主版本号
  - `10`, 次版本号，当前为稳定版本
  - `0`, 修订版本号
- `1160.31.1`，表示发型版本的补丁版本
- `el7`：表示正在使用的内核是 RedHat 7.x / CentOS 7.x 系列发行版专用内核
- `x86_64`：采用的是64位的CPU

## 内核版本分类

1. `mainline`，主线版本。
2. `stable`，稳定版。
3. `longterm`(`Long Term Support`)，长期支持版，长期支持版的内核不再支持时会标记`EOL`（`End of Life`）。
4. `linux-next`，`snapshot`，代码提交周期结束之前生成的快照，用于给Linux代码贡献者们做测试。

## 查看Linux内核版本命令

### `cat /proc/version`

```bash
Linux version 3.10.0-1160.31.1.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) ) #1 SMP Thu Jun 10 13:32:12 UTC 2021
```

### `uname`

```bash
用法：uname [选项]...
输出一组系统信息。如果不跟随选项，则视为只附加-s 选项。

  -a, --all                     以如下次序输出所有信息。其中若-p 和
                                -i 的探测结果不可知则被省略：
  -s, --kernel-name             输出内核名称
  -n, --nodename                输出网络节点上的主机名
  -r, --kernel-release          输出内核发行号
  -v, --kernel-version          输出内核版本
  -m, --machine                 输出主机的硬件架构名称
  -p, --processor               输出处理器类型或"unknown"
  -i, --hardware-platform       输出硬件平台或"unknown"
  -o, --operating-system        输出操作系统名称
      --help                    显示此帮助信息并退出
      --version                 显示版本信息并退出
```
