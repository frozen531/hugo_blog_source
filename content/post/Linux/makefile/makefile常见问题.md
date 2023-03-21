---
title: "Makefile常见问题"
date: 2023-02-12T21:28:47+08:00
draft: true
tags: ["makefile"]
categories: ["Linux"]
---

## 1. *** 遗漏分隔符 。 停止
```bash
[root@bogon 01_hello_world]# make
makefile:2: *** 遗漏分隔符 。 停止。
```
command前一定要是tab不能是空格

## 2. 增加宏但编译未体现
gcc编译不会全部编译，当.o存在时，会比较对应.c和.o的修改时间，如果.c时间早于.o，认为文件没有变化，那么.o也就无需重新编译。

这有时候会导致，当你新增了一个编译宏作用在.c，但是.c之前已按照没有编译宏做了编译，且未做任何修改，那么会发现编译一直报错，这时需要删掉原来的.o重新编译。

## 3. undefined reference to `main'
报错如下：
```bash
[root@bogon add]# make
gcc add.c -o add.o
/usr/lib/gcc/x86_64-redhat-linux/4.4.6/../../../../lib64/crt1.o: In function `_start':
(.text+0x20): undefined reference to `main'
collect2: ld 返回 1
make: *** [TARGET] 错误 1
```

提示找不到`main`，这是因为`gcc add.c -o add.o`是要生成一个可执行文件，可执行意味着里面会包含函数入口`main`（目前并不知道为什么链接不到`main`?）；我的意图是生成`add.o`，应该改为`gcc -c add.c -o add.o`。