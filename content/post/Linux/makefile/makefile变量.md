---
title: "Makefile变量"
date: 2023-02-23T22:30:25+08:00
draft: true
tags: ["makefile"]
categories: ["Linux"]
---


## 1. 变量介绍

- makefile中的变量本质是字符串，用来代表一些文件名列表、编译选项列表、目标文件保存列表等。
- 变量定义。变量可以是空、一项或多项，等号左右两边空格无要求。
- 变量区分大小写，推荐内部定义的一般变量用小写。

### 1.1 自定义变量
- 定义变量（通常小写）
- 变量名=值1 值2 ...
- 使用变量 `$(变量名)`

```bash
#变量赋值并追加
inc_dir := -I./inc
inc_dir += -I../m/

#变量赋值，包含两个.c
src := ./src/a.c ./src/b.c
```

- 变量使用，可以通过`$(name)`和`${name}`

```bash
objs:=$(patsubst %.c, %.o, $(src))
```

### 1.2 赋值shell命令

makefile中可以调用shell命令，可用来初始化文件中的变量和在规则中执行

```bash
#初始化文件中变量
cur = $(shell pwd)

test:
    @echo $(cur)
    date

#结果
[root@localhost 03_make_set]# make
/00_test/03_make_set
date
2023年 02月 25日 星期六 19:45:27 CST
```

### 1.3 自动化变量 $< $@ $? $^

符号 | 说明
--- | ---
`$@`|所有目标文件
`$<`|第一个依赖文件
`$^`|目标依赖的所有文件
`$?`|所有更新过的依赖文件

## 2 变量赋值，四种基本赋值方式

方式 | 说明
---|---
简单赋值 ( := ) | 只对当前语句有效
递归赋值 ( = ) | 所有与该变量 有关的其他变量都受影响
追加赋值 ( += ) | 原变量用空格隔开的方式追加一个新值
条件赋值 ( ?= ) | 如果变量未定义，则使用符号中的值定义变量。如果该变量已经赋值，则该赋值语句无效。

```
x:=foo
y:=$(x)b
x?=new
test:
        @echo "y => $(y)"
        @echo "x => $(x)"
```

## 3. 变量导出

一次执行多个makefile时，如果想让某个变量可在其他makeflie中都可见，可以将变量导出，例如编译工具链。

```bash
export CC = $(CORSS_COMPILE)gcc
```

## 4. 在执行make的脚本中外部传参
```bash
make clean
make CONFIGFILE=$make_config_dir/config.txt WORK_PATH=$work_different_dir
```
这种做法需要注意：如果makefile中`CONFIGFILE`已被赋值，这样传参会覆盖掉原来的值。

有时候我们想脱离脚本单独运行make，这时没有外部入参，makefile中的值缺少赋值，不能运行，可以在makefile中给定默认值。
```bash
ifndef $(WORK_PATH)
    WORK_PATH=/00_test/src
endif

#或者

WORK_PATH?=/00_test/src
```

## 参考链接
1. [Makefile变量的定义和使用](https://blog.csdn.net/qq_42746890/article/details/123580251)