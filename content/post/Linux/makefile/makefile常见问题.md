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