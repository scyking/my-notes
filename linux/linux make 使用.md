# linux make 使用

标签（空格分隔）： linux

---

[toc]

## 概述

### make

> `make` 是一个命令工具，是一个解释 `Makefile` 中指令的命令工具。
> 它可以简化编译过程里面所下达的指令，当执行 `make` 时，`make` 会在当前的目录下搜寻 `Makefile` 这个文本文件，执行对应的操作。
> `make` 会自动的判别原始码是否经过变动了，而自动更新执行档。

### Makefile

> `Makefile` 其实就是一个文档，里面定义了一系列的规则指定哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，它记录了原始码如何编译的详细信息。
> `Makefile` 一旦写好，只需要一个`make` 命令，整个工程完全自动编译，极大的提高了软件开发的效率。

### 安装 make

```bash
yum install gcc automake autoconf libtool make
```

## Makefile 规则

### 主要规则

1. 显示规则 : 说明如何生成一个或多个目标文件(包括 生成的文件, 文件的依赖文件, 生成的命令)
1. 隐晦规则 : make的自动推导功能所执行的规则
1. 变量定义 : Makefile中定义的变量
1. 文件指示 : Makefile中引用其他Makefile; 指定Makefile中有效部分; 定义一个多行命令
1. 注释     : Makefile只有行注释 "`#`", 如果要使用或者输出"`#`"字符, 需要进行转义, "`\#`"

### 基本格式

```bash
target ... : prerequisites ...
    command
    ...
# 或者
target ... : prerequisites ; command
    command
    ...
# command太长, 可以用 "\" 作为换行符
```

- `target`        ：目标文件, 可以是 Object File, 也可以是可执行文件
- `prerequisites` ：生成 `target` 所需要的文件或者目标
- `command`       ：make需要执行的命令 (任意的shell命令), Makefile中的命令必须以 `[tab]` 开头

### 通配符

- `*`     : 表示任意一个或多个字符
- `?`     : 表示任意一个字符
- `[...]` : ex. `[abcd]` 表示`a,b,c,d`中任意一个字符；`[^abcd]`表示除`a,b,c,d`以外的字符；`[0-9]`表示 `0~9`中任意一个数字
- `~`     : 表示用户的`home`目录

### 路径搜索

> 当一个Makefile中涉及到大量源文件时，为便于编译时查找，使用变量`VPATH`。
> 指定了 `VPATH` 之后, 如果当前目录中没有找到相应文件或依赖的文件, Makefile 回到 `VPATH` 指定的路径中再去查找。

- `vpath <directories>`: 当前目录中找不到文件时, 就从`<directories>`中搜索
- `vpath <pattern> <directories>`: 符合`<pattern>`格式的文件, 就从`<directories>`中搜索
- `vpath <pattern>`: 清除符合`<pattern>`格式的文件搜索路径
- `vpath`: 清除所有已经设置好的文件路径

```bash
# 示例1 - 当前目录中找不到文件时, 按顺序从 src目录 ../parent-dir目录中查找文件
VPATH src:../parent-dir   

# 示例2 - .h结尾的文件都从 ./header 目录中查找
VPATH %.h ./header

# 示例3 - 清除示例2中设置的规则
VPATH %.h

# 示例4 - 清除所有VPATH的设置
VPATH
```

## Makefile 变量

### 变量定义

- `:=`：只能使用前面定义好的变量
- `=`：可以使用后面定义的变量

```bash
OBJS = programA.o programB.o
OBJS-ADD = $(OBJS) programC.o
# 或者
OBJS := programA.o programB.o
OBJS-ADD := $(OBJS) programC.o
```

### 变量替换

```bash
# Makefile内容
SRCS := programA.c programB.c programC.c
OBJS := $(SRCS:%.c=%.o)

all:
    @echo "SRCS: " $(SRCS)
    @echo "OBJS: " $(OBJS)

# bash中运行make
$ make
SRCS:  programA.c programB.c programC.c
OBJS:  programA.o programB.o programC.o
```

### 变量追加

```bash
# Makefile内容
SRCS := programA.c programB.c programC.c
SRCS += programD.c

all:
    @echo "SRCS: " $(SRCS)

# bash中运行make
$ make
SRCS:  programA.c programB.c programC.c programD.c
```

### 变量覆盖

> 作用是使 Makefile中定义的变量能够覆盖 make 命令参数中指定的变量

- `override <variable> = <value>`
- `override <variable> := <value>`
- `override <variable> += <value>`

```bash
# Makefile内容 (没有用override)
SRCS := programA.c programB.c programC.c

all:
    @echo "SRCS: " $(SRCS)

# bash中运行make
$ make SRCS=nothing
SRCS:  nothing

#################################################

# Makefile内容 (用override)
override SRCS := programA.c programB.c programC.c

all:
    @echo "SRCS: " $(SRCS)

# bash中运行make
$ make SRCS=nothing
SRCS:  programA.c programB.c programC.c
```

### 目标变量

> 作用是使变量的作用域仅限于这个目标(target), 而不像之前例子中定义的变量, 对整个Makefile都有效。

- `<target ...>`: `<variable-assignment>`
- `<target ...>`: `override <variable-assignment>`

```bash
# Makefile 内容
SRCS := programA.c programB.c programC.c

target1: TARGET1-SRCS := programD.c
target1:
    @echo "SRCS: " $(SRCS)
    @echo "SRCS: " $(TARGET1-SRCS)

target2:
    @echo "SRCS: " $(SRCS)
    @echo "SRCS: " $(TARGET1-SRCS)

# bash中执行make
$ make target1
SRCS:  programA.c programB.c programC.c
SRCS:  programD.c

$ make target2     <-- target2中显示不了 $(TARGET1-SRCS)
SRCS:  programA.c programB.c programC.c
SRCS:
```

### 隐含规则中的变量

> 写shell时，可直接使用这些变量。

#### 命令变量

|变量名|含义|
|---|---|
|`RM` |`rm -f`|
|`AR` |`ar`|
|`CC` |`cc`|
|`CXX` |`g++`|

#### 参数变量

|变量名|含义|
|---|---|
|`ARFLAGS` |AR命令的参数
|`CFLAGS` |C语言编译器的参数
|`CXXFLAGS` |C++语言编译器的参数

#### 自动变量

|变量名|含义|
|---|---|
|`$@` |目标集合
|`$%` |当目标是函数库文件时, 表示其中的目标文件名
|`$<` |第一个依赖目标. 如果依赖目标是多个, 逐个表示依赖目标
|`$?` |比目标新的依赖目标的集合
|`$^` |所有依赖目标的集合, 会去除重复的依赖目标
|`$+` |所有依赖目标的集合, 不会去除重复的依赖目标
|`$*` |这个是GNU make特有的, 其它的make不一定支持

## Makefile 命令前缀

> Makefile 中书写shell命令时可以加2种前缀 @ 和 -, 或者不用前缀。

- 不用前缀 : 输出执行的命令以及命令执行的结果, 出错的话停止执行
- 前缀 `@` : 只输出命令执行的结果, 出错的话停止执行
- 前缀 `-` : 命令执行有错的话, 忽略错误, 继续执行

```bash
# Makefile 内容 (不用前缀)
all:
    echo "没有前缀"
    cat this_file_not_exist
    echo "错误之后的命令"       <-- 这条命令不会被执行

# bash中执行 make
$ make
echo "没有前缀"             <-- 命令本身显示出来
没有前缀                    <-- 命令执行结果显示出来
cat this_file_not_exist
cat: this_file_not_exist: No such file or directory
make: *** [all] Error 1

###########################################################

# Makefile 内容 (前缀 @)
all:
    @echo "没有前缀"
    @cat this_file_not_exist
    @echo "错误之后的命令"       <-- 这条命令不会被执行

# bash中执行 make
$ make
没有前缀                         <-- 只有命令执行的结果, 不显示命令本身
cat: this_file_not_exist: No such file or directory
make: *** [all] Error 1

###########################################################

# Makefile 内容 (前缀 -)
all:
    -echo "没有前缀"
    -cat this_file_not_exist
    -echo "错误之后的命令"       <-- 这条命令会被执行

# bash中执行 make
$ make
echo "没有前缀"             <-- 命令本身显示出来
没有前缀                    <-- 命令执行结果显示出来
cat this_file_not_exist
cat: this_file_not_exist: No such file or directory
make: [all] Error 1 (ignored)
echo "错误之后的命令"       <-- 出错之后的命令也会显示
错误之后的命令              <-- 出错之后的命令也会执行
```

## Makefile 伪目标

> 目标并非实际文件时，它就成为了所谓的“伪目标”。当仅执行`make`命令时，伪目标会被忽略，只有在显式指定为目标时，伪目标的命令才会被执行。

```bash
# 可以使用.PHONY将某一目标显式指定为伪目标。
.PHONY: clean
clean:
    rm -f *.o
```

### 约定俗成的伪目标

|伪目标|含义|
|---|---|
|`all`         |所有目标的目标，其功能一般是编译所有的目标
|`clean`     |删除所有被make创建的文件
|`install`     |安装已编译好的程序，其实就是把目标可执行文件拷贝到指定的目录中去
|`print`     |列出改变过的源文件
|`tar`         |把源程序打包备份. 也就是一个tar文件
|`dist`         |创建一个压缩文件, 一般是把tar文件压成Z文件. 或是gz文件
|`TAGS`         |更新所有的目标, 以备完整地重编译使用
|`check`/`est` |一般用来测试Makefile的流程

## 引用其他Makefile

### 语法

```bash
include <filename>  (filename 可以包含通配符和路径)
```

### 示例

```bash
# Makefile 内容
all:
    @echo "主 Makefile begin"
    @make other-all
    @echo "主 Makefile end"

include ./other/Makefile

# ./other/Makefile 内容
other-all:
    @echo "other Makefile begin"
    @echo "other Makefile end"

# bash中执行 make
$ ll
total 20K
-rw-r--r-- 1 wangyubin wangyubin  125 Sep 23 16:13 Makefile
-rw-r--r-- 1 wangyubin wangyubin  11K Sep 23 16:15 Makefile.org   <-- 这个文件不用管
drwxr-xr-x 2 wangyubin wangyubin 4.0K Sep 23 16:11 other
$ ll other/
total 4.0K
-rw-r--r-- 1 wangyubin wangyubin 71 Sep 23 16:11 Makefile

$ make
主 Makefile begin
make[1]: Entering directory `/path/to/test/Makefile'
other Makefile begin
other Makefile end
make[1]: Leaving directory `/path/to/test/Makefile'
主 Makefile end
```

## Makefile 退出码

- `0` : 表示成功执行
- `1` : 表示make命令出现了错误
- `2` : 使用了 "`-q`" 选项, 并且make使得一些目标不需要更新

## 指定Makefile

> 默认执行 make 命令时, GNU make在当前目录下依次搜索下面3个文件 "GNUMakefile", "Makefile", "Makefile"。

```bash
# Makefile文件名改为 MyMake, 内容
target1:
    @echo "target [1]  begin"
    @echo "target [1]  end"

target2:
    @echo "target [2]  begin"
    @echo "target [2]  end"

# bash 中执行 make
$ ls
Makefile
$ mv Makefile MyMake
$ ls
MyMake
$ make                     <-- 找不到默认的 Makefile
make: *** No targets specified and no Makefile found.  Stop.
$ make -f MyMake           <-- 指定特定的Makefile
target [1]  begin
target [1]  end
$ make -f MyMake target2   <-- 指定特定的目标(target)
target [2]  begin
target [2]  end
```

## Makefile高阶语法

### 嵌套Makefile

> 是引用其它Makefile的另一种写法，并且可以向引用的其它 Makefile 传递参数。

`export` 语法格式：

- `export variable = value`
- `export variable := value`
- `export variable += value`

```bash
# Makefile 内容
export VALUE1 := export.c    <-- 用了 export, 此变量能够传递到 ./other/Makefile 中
VALUE2 := no-export.c        <-- 此变量不能传递到 ./other/Makefile 中

all:
    @echo "主 Makefile begin"
    @cd ./other && make
    @echo "主 Makefile end"


# ./other/Makefile 内容
other-all:
    @echo "other Makefile begin"
    @echo "VALUE1: " $(VALUE1)
    @echo "VALUE2: " $(VALUE2)
    @echo "other Makefile end"

# bash中执行 make
$ make
主 Makefile begin
make[1]: Entering directory `/path/to/test/Makefile/other'
other Makefile begin
VALUE1:  export.c        <-- VALUE1 传递成功
VALUE2:                  <-- VALUE2 传递失败
other Makefile end
make[1]: Leaving directory `/path/to/test/Makefile/other'
主 Makefile end
```

### 定义命令包

> 命令包有点像是个函数, 将连续的相同的命令合成一条, 减少 Makefile 中的代码量, 便于以后维护。

语法：

```bash
define <command-name>
command
...
endef
```

示例：

```bash
# Makefile 内容
define run-hello-Makefile
@echo -n "Hello"
@echo " Makefile!"
@echo "这里可以执行多条 Shell 命令!"
endef

all:
    $(run-hello-Makefile)


# bash 中运行make
$ make
Hello Makefile!
这里可以执行多条 Shell 命令!
```

### 条件判断

- `ifeq`
- `ifneq`
- `ifdef`
- `ifndef`

```bash
<conditional-directive>
<text-if-true>
endif

# 或者
<conditional-directive>
<text-if-true>
else
<text-if-false>
endif
```

### Makefile 中函数

Makefile函数的基本使用模式如下：

```bash
$(<func_name> <param1>,<param2>,...)
```

#### 字符串函数

##### 字符串替换函数：`subst`

> `$(subst <from>,<to>,<text>)`。
> 把字符串 `<text>` 中的 `<from>` 替换为 `<to>`，返回替换过的字符串。

```bash
# Makefile 内容
all:
    @echo $(subst t,e,maktfilt)  <-- 将t替换为e

# bash 中执行 make
$ make
Makefile
```

##### 模式字符串替换函数：`patsubst`

> `$(patsubst <pattern>,<replacement>,<text>)`。
> 查找`<text>`中的单词(单词以"空格", "tab", "换行"来分割) 是否符合 `<pattern>`, 符合的话, 用 `<replacement>` 替代。

```bash
# Makefile 内容
all:
    @echo $(patsubst %.c,%.o,programA.c programB.c)

# bash 中执行 make
$ make
programA.o programB.o
```

##### 去空格函数：`strip`

> `$(strip <string>)`。
> 去掉 `<string>` 字符串中开头和结尾的空字符。

```bash
# Makefile 内容
VAL := "       aa  bb  cc "

all:
    @echo "去除空格前: " $(VAL)
    @echo "去除空格后: " $(strip $(VAL))

# bash 中执行 make
$ make
去除空格前:         aa  bb  cc 
去除空格后:   aa bb cc
```

##### 查找字符串函数：`findstring`

> `$(findstring <find>,<in>)`。
> 在字符串 `<in>` 中查找 `<find>` 字符串。
> 如果找到, 返回 `<find>` 字符串,  否则返回空字符串。

```bash
# Makefile 内容
VAL := "       aa  bb  cc "

all:
    @echo $(findstring aa,$(VAL))
    @echo $(findstring ab,$(VAL))

# bash 中执行 make
$ make
aa
```

##### 过滤函数：`filter`

> `$(filter <pattern...>,<text>)`。
> 以 `<pattern>` 模式过滤字符串 `<text>`, **保留** 符合模式 `<pattern>` 的单词, 可以有多个模式。

```bash
# Makefile 内容
all:
    @echo $(filter %.o %.a,program.c program.o program.a)


# bash 中执行 make
$ make
program.o program.a
```

##### 反过滤函数：`filter-out`

> `$(filter-out <pattern...>,<text>)`。
> 以 `<pattern>` 模式过滤字符串 `<text>`, **去除** 符合模式 `<pattern>` 的单词, 可以有多个模式。

```bash
# Makefile 内容
all:
    @echo $(filter-out %.o %.a,program.c program.o program.a)

# bash 中执行 make
$ make
program.c
```

##### 排序函数：`sort`

> `$(sort <list>)`。
> 给字符串 `<list>` 中的单词排序 (升序)，返回排序后的字符串。

```bash
# Makefile 内容
all:
    @echo $(sort bac abc acb cab)

# bash 中执行 make
$ make
abc acb bac cab
```

##### 取单词函数：`word`

> `$(word <n>,<text>)`。
> 取字符串 `<text>` 中的 第`<n>`个单词 (n从1开始)，返回`<text>` 中的第`<n>`个单词。
> 如果`<n>`比`<text>`中单词个数要大， 则返回空字符串。

```bash
# Makefile 内容
all:
    @echo $(word 1,aa bb cc dd)
    @echo $(word 5,aa bb cc dd)
    @echo $(word 4,aa bb cc dd)

# bash 中执行 make
$ make
aa

dd
```

##### 取单词串函数：`wordlist`

> `$(wordlist <s>,<e>,<text>)`。
> 从字符串`<text>`中取从`<s>`开始到`<e>`的单词串。`<s>`和`<e>`是一个数字。

```bash
# Makefile 内容
all:
    @echo $(wordlist 1,3,aa bb cc dd)
    @echo $(word 5,6,aa bb cc dd)
    @echo $(word 2,5,aa bb cc dd)


# bash 中执行 make
$ make
aa bb cc

bb
```

##### 单词个数统计函数：`words`

> `$(words <text>)`。
> 统计字符串 `<text>` 中单词的个数。

```bash
# Makefile 内容

all:
    @echo $(words aa bb cc dd)
    @echo $(words aabbccdd)
    @echo $(words )

# bash 中执行 make
$ make
4
1
0
```

##### 首单词函数：`firstword`

> `$(firstword <text>)`。
> 取字符串 `<text>` 中的第一个单词。

```bash
# Makefile 内容
all:
    @echo $(firstword aa bb cc dd)
    @echo $(firstword aabbccdd)
    @echo $(firstword )

# bash 中执行 make
$ make
aa
aabbccdd
```

#### 文件名函数

##### 取目录函数：`dir`

> `$(dir <names...>)`。
> 从文件名序列 `<names>` 中取出目录部分。

```bash
# Makefile 内容
all:
    @echo $(dir /home/a.c ./bb.c ../c.c d.c)


# bash 中执行 make
$ make
/home/ ./ ../ ./
```

##### 取文件函数：`notdir`

> `$(notdir <names...>)`。
> 从文件名序列 `<names>` 中取出非目录部分。

```bash
# Makefile 内容
all:
    @echo $(notdir /home/a.c ./bb.c ../c.c d.c)

# bash 中执行 make
$ make
a.c bb.c c.c d.c
```

##### 取后缀函数：`suffix`

> `$(suffix <names...>)`。
> 从文件名序列 `<names>` 中取出各个文件名的后缀。

```bash
# Makefile 内容
all:
    @echo $(suffix /home/a.c ./b.o ../c.a d)

# bash 中执行 make
$ make
.c .o .a
```

##### 取前缀函数：`basename`

> `$(basename <names...>)`。
> 从文件名序列 `<names>` 中取出各个文件名的前缀。

```bash
# Makefile 内容
all:
    @echo $(basename /home/a.c ./b.o ../c.a /home/.d .e)


# bash 中执行 make
$ make
/home/a ./b ../c /home/
```

##### 加后缀函数：`addsuffix`

> `$(addsuffix <suffix>,<names...>)`。
> 把后缀 `<suffix>` 加到 `<names>` 中的每个单词后面。

```bash
# Makefile 内容
all:
    @echo $(addsuffix .c,/home/a b ./c.o ../d.c)


# bash 中执行 make
$ make
/home/a.c b.c ./c.o.c ../d.c.c
```

##### 加前缀函数：`addprefix`

> `$(addprefix <prefix>,<names...>)`。
> 把前缀 `<prefix>` 加到 `<names>` 中的每个单词前面。

```bash
# Makefile 内容
all:
    @echo $(addprefix test_,/home/a.c b.c ./d.c)

# bash 中执行 make
$ make
test_/home/a.c test_b.c test_./d.c
```

##### 连接函数：`join`

> `$(join <list1>,<list2>)`。
> `<list2>` 中对应的单词加到 `<list1>` 后面。

```bash
# Makefile 内容
all:
    @echo $(join a b c d,1 2 3 4)
    @echo $(join a b c d,1 2 3 4 5)
    @echo $(join a b c d e,1 2 3 4)

# bash 中执行 make
$ make
a1 b2 c3 d4
a1 b2 c3 d4 5
a1 b2 c3 d4 e
```

#### `foreach`

> `$(foreach <var>,<list>,<text>)`。

```bash
# Makefile 内容
targets := a b c d
objects := $(foreach i,$(targets),$(i).o)

all:
    @echo $(targets)
    @echo $(objects)

# bash 中执行 make
$ make
a b c d
a.o b.o c.o d.o
```

#### `if`

> `$(if <condition>,<then-part>)`或`$(if <condition>,<then-part>,<else-part>)`。
> 这里的if是个函数, 和前面的条件判断不一样, 前面的条件判断属于Makefile的关键字。

```bash
# Makefile 内容
val := a
objects := $(if $(val),$(val).o,nothing)
no-objects := $(if $(no-val),$(val).o,nothing)

all:
    @echo $(objects)
    @echo $(no-objects)

# bash 中执行 make
$ make
a.o
nothing
```

#### `call`

> `$(call <expression>,<parm1>,<parm2>,<parm3>...)`。
> 创建新的参数化函数。

```bash
# Makefile 内容
log = "====debug====" $(1) "====end===="

all:
    @echo $(call log,"正在 Make")

# bash 中执行 make
$ make
====debug==== 正在 Make ====end====
```

#### `origin`

> `$(origin <variable>)`。
> 判断变量来源。

##### 类型

|类型|含义|
|---|---|
|`undefined`     |`<variable>` 没有定义过
|`default`         |`<variable>` 是个默认的定义, 比如 CC 变量
|`environment`     |`<variable>` 是个环境变量, 并且 make时没有使用 -e 参数
|`file`             |`<variable>` 定义在Makefile中
|`command line`     |`<variable>` 定义在命令行中
|`override`         |`<variable>` 被 override 重新定义过
|`automatic`     |`<variable>` 是自动化变量

##### 判断变量来源

```bash
# Makefile 内容
val-in-file := test-file
override val-override := test-override

all:
    @echo $(origin not-define)    # not-define 没有定义
    @echo $(origin CC)            # CC 是Makefile默认定义的变量
    @echo $(origin PATH)          # PATH 是 bash 环境变量
    @echo $(origin val-in-file)   # 此Makefile中定义的变量
    @echo $(origin val-in-cmd)    # 这个变量会加在 make 的参数中
    @echo $(origin val-override)  # 此Makefile中定义的override变量
    @echo $(origin @)             # 自动变量, 具体前面的介绍

# bash 中执行 make
$ make val-in-cmd=val-cmd
undefined
default
environment
file
command line
override
automatic
```

#### `shell`

> `$(shell <shell command>)`。
> 它的作用就是执行一个shell命令，并将shell命令的结果作为函数的返回。

#### make 控制函数

##### 产生一个致命错误

> `$(error <text ...>)`。
> 输出错误信息, 停止Makefile的运行。

```bash
# Makefile 内容
all:
    $(error there is an error!)
    @echo "这里不会执行!"

# bash 中执行 make
$ make
Makefile:2: *** there is an error!.  Stop.
```

##### 输出警告

> `$(warning <text ...>)`
> 输出警告信息, Makefile继续运行。

```bash
# Makefile 内容
all:
    $(warning there is an warning!)
    @echo "这里会执行!"

# bash 中执行 make
$ make
Makefile:2: there is an warning!
这里会执行!
```
