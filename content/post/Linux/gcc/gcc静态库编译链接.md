---
title: "Gcc静态库编译链接"
date: 2023-03-01T22:40:32+08:00
draft: true
tags: ["gcc"]
categories: [Linux]
---


静态库实际上是一组目标文件的集合，即很多目标文件经过压缩打包后形成的一个文件。--- 《程序员的自我修养》

## 1. 示例代码



## 2. 生成静态库
库的链接有两种方式：
1. -L路径 -l库名：`-L./human/ -lhuman `
2. 直接路径链接加库名：`./human/libhuman.a`

- main.c

```c
#include <stdio.h>

extern int process(int a);

int main()
{
    process(3);
    return 0;
}
```

## 3. 链接静态库

```bash
TARGET:main.c
	gcc -c main.c
	gcc -o app_static main.o ./human/libhuman.a 
```
注意：
> 1. 编译讲究先后顺序，需要`main.o`在`./human/libhuman.a`前，否则会出现未定义的引用，报错如下：***需确认书上的解释***
> 2. 对于库链接的两种方式， `app_static`均可正常运行

```bash
gcc -c main.c
gcc -o app_static ./human/libhuman.a main.o
main.o: In function `main':
main.c:(.text+0xa): undefined reference to `process'
collect2: error: ld returned 1 exit status
makefile:2: recipe for target 'TARGET' failed
make: *** [TARGET] Error 1
```

## 4. 运行

```bash
[root@localhost 01_newlib]# ./app_static
human:process 3
play:human
```

合并静态库有两种方法：
1. 将所有.a解压为众多的.o，然后合为一个.a


2. 直接将多个.a合为一个

```bash
LIB_TARGET := 
TMP_MRI := merge.mri

$(TARGET): $(COBJS)
	touch $(TMP_MRI)
	echo "create $(TARGET)" >> $(TMP_MRI)
	@$(foreach lib, $(LIBS), echo "addlib $(lib)" >> $(TMP_MRI);)
	echo "addlib $(TMP_LIB)" >> $(TMP_MRI)
	echo save >> $(TMP_MRI)
	echo end >> $(TMP_MRI)
	ar -M < $(TMP_MRI)
```
