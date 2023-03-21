---
title: "Gcc静态库编译链接"
date: 2023-03-01T22:40:32+08:00
draft: true
tags: ["gcc"]
categories: [Linux]
---


静态库实际上是一组目标文件的集合，即很多目标文件`.o`经过压缩打包后形成的一个文件。--------- 《程序员的自我修养 4.5静态库链接》

## 1. 示例代码
构建如下代码结构。
```bash
[root@bogon 05_static]# tree
.
├── add
│   ├── add.c
│   ├── libadd.a
│   └── makefile
├── main.c
├── makefile
├── mul
│   ├── div.c
│   ├── libmul_div.a
│   ├── makefile
│   └── mul.c
└── sub
    ├── libsub.a
    ├── makefile
    └── sub.c
```

## 2. 生成静态库



### `add`

这里makefile用了add文件名，在迁移上局限性很高。

```bash
// add.c
#include <stdio.h>

void add(int a, int b)
{
        printf("%d + %d = %d \n", a, b, a + b);
}

// makefile
TARGET := libadd.a

TARGET : 
        gcc -c add.c -o add.o
        ar rc $(TARGET) add.o

clean:
        rm -f add.o
```

### `sub`

使用函数`wildcard`取消了对文件名的依赖，只有target需要指定，并使用了通配符，简化操作。这里需要注意：`$@`是目标，从下面`makefile`可以看出目标即`main`这个变量名。

```bash
// sub.c
void sub(int a, int b)
{
        printf("%d - %d = %d \n", a, b, a - b);
}

// makefile
SOURCE := $(wildcard ./*.c)
OBJECT := $(patsubst %.c, %.o, $(SOURCE))

TARGET := libsub.a

main : $(OBJECT) 
        echo target:$@
        ar rc $(TARGET) $^


%.o : %.c
        gcc -c $^ -o $@

clean:
        rm -f $(TARGET) $(OBJECT)
```

### `mul`

该目录下包含两个.c，makefile同上。

```bash
// mul.c
#include <stdio.h>

void mul(int a, int b)
{
        printf("%d * %d = %d \n", a, b, a*b);
}

// div.c
#include <stdio.h>

void div(int a, int b)
{
        printf("%d / %d = %d \n", a, b, a/b);
}

// makefile
SOURCE := $(wildcard ./*.c)
OBJECT := $(patsubst %.c, %.o, $(SOURCE))

TARGET := libmul_div.a

$(TARGET) : $(OBJECT) 
        echo target:$(TARGET)
        ar rc $@ $^


%.o : %.c
        gcc -c $^ -o $@

CLEAN:
        rm -f $(TARGET) $(OBJECT)
```

## 3. 链接静态库

### `main`

```bash
// main.c
int main()
{
        add(3, 4);
        sub(5, 6);
        mul(7, 8);
        div(6, 4);
        return 0;
}
```

库的链接有两种方式：
1. -L路径 -l库名：`-L./human/ -lhuman `

```bash
// makefile
libs := -L./add/ -ladd
libs += -L./sub/ -lsub
libs += -L./mul/ -lmul_div

target := app

$(target) : 
        gcc -c main.c -o main.o
        gcc -o $@ main.o $(libs)

clean:
        rm -f *.o $(target)
```

2. 直接路径链接加库名：`./human/libhuman.a`

```bash
// makefile
libs := ./add/libadd.a
libs += ./sub/libsub.a
libs += ./mul/libmul_div.a

target := app

$(target) : 
        gcc -c main.c -o main.o
        gcc -o $@ main.o $(libs)

clean:
        rm -f *.o $(target)
```


注意：
> 1. 编译讲究先后顺序，`gcc -o $@ main.o $(libs)`，需要`main.o`在`./human/libhuman.a`前，否则会出现未定义的引用，报错如下：***需确认书上的解释***

```bash
[root@bogon 05_static]# make -f makefile_L
gcc -c main.c -o main.o
gcc -o app -L./add/ -ladd -L ./sub/ -lsub -L ./mul/ -lmul_div main.o
main.o: In function `main':
main.c:(.text+0x14): undefined reference to `add'
main.c:(.text+0x28): undefined reference to `sub'
main.c:(.text+0x3c): undefined reference to `mul'
collect2: ld 返回 1
make: *** [app] 错误 1
```

> 2. 对于库链接的两种方式， `app`均可正常运行

## 4. 运行

```bash
[root@localhost 01_newlib]# ./app
3 + 4 = 7 
5 - 6 = -1 
7 * 8 = 56 
6 / 4 = 1
```

## 5. 合并静态库

合并静态库有两种方法：
1. 将所有.a解压为众多的.o，然后合为一个.a
2. 直接将多个.a合为一个

### 解压成.o后合并



### 直接合并多个.a

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
