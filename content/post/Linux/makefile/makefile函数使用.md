---
title: "Makefile函数使用"
date: 2023-02-23T22:26:51+08:00
draft: true
tags: ["makefile"]
categories: ["Linux"]
---

## 函数
函数的调用语法：

```bash
$(<function> <arguments>)

//举例如下：
$(wildcard *.c)
```

### wildcard 扩展通配符函数
常见的通配符有：

通配符 | 含义
|---|---|
|* |  匹配任意长度的任意字符，可以是0个|
|？ |  匹配任意单个字符，必须是1个|
|[] | 匹配指定字符范围内的任意单个字符|

在`Makefile`规则中，通配符会被自动展开。但在变量的定义和函数引用时，通配符将失效。这种情况下，如果需要通配符有效，就需要使用函数“wildcard”。

用法：`$(wildcard RATTERN...)`

在Makefile中，它被展开为已经存在的、使用空格分开的、匹配此模式的所有文件列表。

```powershell
# 当前列表中的所有.c文件
$(wildcard *.c)

# 获取$(FILE_DIR)/src中所有.c文件
SOURCES += $(wildcard $(FILE_DIR)/src/*.c)
```

### patsubst 替换通配符函数

```powershell
# 将所有文件名后缀.c替换为.o，这些.c文件来自于后面的位置
OBJECTS := $(patsubst %.c,%.o,$(SOURCES))
```

### notdir 去除路径

```powershell
@echo cut $(notdir $(SOURCE))
```

### foreach 循环

```bash
#基本用法：将list列表中的值一个个取出给到var，var再去执行text，对应将产生多个text的结果
$(foreach <var>,<list>,<text>)

#使用
names := a b c d
files := $(foreach n,$(names),$(n).o)

test:
        @echo $(files)

#结果
[root@localhost 03_make_set]# make
a.o b.o c.o d.o
```

### basename 取前缀
```bash
#基本用法
$(basename <names...>)

#使用
SRC := src/main.c src/hello.c
OBJ := $(basename $(SRC)) 
all:
	@echo "$(OBJ)"

#结果
[root@localhost 03_make_set]# make
src/main src/hello
```


