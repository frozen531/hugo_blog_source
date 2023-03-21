---
title: "Gcc编译链接选项_section减少可执行程序大小"
date: 2023-03-15T07:44:46+08:00
draft: true
tags: ["gcc"]
categories: [Linux]
---

## 背景

静态库是多个目标文件的集合，链接生成可执行文件时，链接器以目标文件为单位。即如果很多函数都放到一个目标文件中，当该文件被链接，会有很多不用的函数一起链入，导致可执行文件增大。处理方式有两种：

1. 同C库函数操作，尽量每个目标文件中只含有一个函数，但这个会导致文件数增多；
2. 使用如下编译链接选项：

```bash
CFLAGS += -ffunction-sections -fdata-sections
LDFLAGS += -Wl,--gc-sections
```

## 编译选项示例解析

`main.c`中放入多个函数和变量，`main()`只调用其中几个；`makefile`中按`CFLAGS`和`LDFLAGS`做组合，对比结果。

### main.c

```c
#include <stdio.h>

int a = 1;
static b = 2;
int c = 3;

int fun_0(void)
{
    printf("%s: %d\n", __FUNCTION__, __LINE__);
    return 0;
}

int fun_1(void)
{
    printf("%s: %d\n", __FUNCTION__, __LINE__);
    return 0;
}

int fun_2(void)
{
    printf("%s: %d\n", __FUNCTION__, __LINE__);
    return 0;
}

int fun_3(void)
{
    printf("%s: %d\n", __FUNCTION__, __LINE__);
    return 0;
}

void main(void)
{
        fun_0();
        fun_3();
        printf("c = %d \n", c);
}
```

### makefile

```bash
CFLAGS += -ffunction-sections -fdata-sections
LDFLAGS += -Wl,--gc-sections

main_section:
        gcc $(CFLAGS) -c main.c -o main_section.o
        gcc $(LDFLAGS) -o $@ main_section.o

main_normal:
        gcc -c main.c -o main_normal.o
        gcc -o $@ main_normal.o

main_section_cflags:
        gcc $(CFLAGS) -c main.c -o main_section_cflags.o
        gcc -o $@ main_section_cflags.o

main_section_ldflags:
        gcc -c main.c -o main_section_ldflags.o
        gcc $(LDFLAGS) -o $@ main_section_ldflags.o

clean:
        rm -f *.o main_section main_normal main_section_cflags main_section_ldflags
```

## 结果比较

gcc链接操作以`section`为最小处理单元，只要`section`中某个符号被引用，该`section`就会被加入可执行程序中。

### 查看生成的*.o大小

```bash
[root@localhost 06_section]# ls -l *.o
-rw-r--r--. 1 root root 2672 3月  16 03:59 main_normal.o
-rw-r--r--. 1 root root 4176 3月  16 03:58 main_section.o
```

以上可以看出：使用`-ffunction-sections -fdata-sections`后，目标文件会比之前变大

### readelf -t查看目标文件

- main_normal.o

```bash
[root@localhost 06_section]# readelf -t main_normal.o
There are 13 section headers, starting at offset 0x290:

Section Headers:
  [Nr] Name
       Type              Address          Offset            Link
       Size              EntSize          Info              Align
       Flags
  [ 0] 
       NULL             0000000000000000  0000000000000000  0
       0000000000000000 0000000000000000  0                 0
       [0000000000000000]: 
  [ 1] .text
       PROGBITS         0000000000000000  0000000000000040  0
       00000000000000c6 0000000000000000  0                 4
       [0000000000000006]: ALLOC, EXEC
  [ 2] .rela.text
       RELA             0000000000000000  0000000000000860  11
       0000000000000198 0000000000000018  1                 8
       [0000000000000000]: 
  [ 3] .data
       PROGBITS         0000000000000000  0000000000000108  0
       000000000000000c 0000000000000000  0                 4
       [0000000000000003]: WRITE, ALLOC
  [ 4] .bss
       NOBITS           0000000000000000  0000000000000114  0
       0000000000000000 0000000000000000  0                 4
       [0000000000000003]: WRITE, ALLOC
  [ 5] .rodata
       PROGBITS         0000000000000000  0000000000000114  0
       0000000000000029 0000000000000000  0                 1
       [0000000000000002]: ALLOC
  [ 6] .comment
       PROGBITS         0000000000000000  000000000000013d  0
       000000000000002d 0000000000000001  0                 1
       [0000000000000030]: MERGE, STRINGS
  [ 7] .note.GNU-stack
       PROGBITS         0000000000000000  000000000000016a  0
       0000000000000000 0000000000000000  0                 1
       [0000000000000000]: 
  [ 8] .eh_frame
       PROGBITS         0000000000000000  0000000000000170  0
       00000000000000b8 0000000000000000  0                 8
       [0000000000000002]: ALLOC
  [ 9] .rela.eh_frame
       RELA             0000000000000000  00000000000009f8  11
       0000000000000078 0000000000000018  8                 8
       [0000000000000000]: 
  [10] .shstrtab
       STRTAB           0000000000000000  0000000000000228  0
       0000000000000061 0000000000000000  0                 1
       [0000000000000000]: 
  [11] .symtab
       SYMTAB           0000000000000000  00000000000005d0  12
       0000000000000210 0000000000000018  14                8
       [0000000000000000]: 
  [12] .strtab
       STRTAB           0000000000000000  00000000000007e0  0
       000000000000007a 0000000000000000  0                 1
       [0000000000000000]: 
```

- main_section.o，这个太长，通过grep搜索

```bash
[root@localhost 06_section]# readelf -t main_section.o | grep fun
  [ 8] .text.fun_0
  [ 9] .rela.text.fun_0
  [10] .text.fun_1
  [11] .rela.text.fun_1
  [12] .text.fun_2
  [13] .rela.text.fun_2
  [14] .text.fun_3
  [15] .rela.text.fun_3

[root@localhost 06_section]# readelf -t main_section.o | grep data
  [ 2] .data
  [ 4] .data.a
  [ 5] .data.b
  [ 6] .data.c
  [ 7] .rodata
  [18] .rodata.__FUNCTION__.2072
  [19] .rodata.__FUNCTION__.2066
  [20] .rodata.__FUNCTION__.2060
  [21] .rodata.__FUNCTION__.2054
```

可以看出，`main_section.o`相比`main_normal.o`的信息，每个函数和数据都成为了独立的`section`，而`main_normal.o`中函数和数据都只在同一个`.text`中，这就是`main_section.o`变大的原因。

### 查看可执行文件大小

```bash
[root@localhost 06_section]# ls -l
总用量 40
-rw-r--r--. 1 root root  468 3月  15 16:25 main.c
-rwxr-xr-x. 1 root root 7208 3月  16 03:59 main_normal
-rwxr-xr-x. 1 root root 6528 3月  16 03:58 main_section
-rwxr-xr-x. 1 root root 7208 3月  16 03:59 main_section_cflags
-rwxr-xr-x. 1 root root 6908 3月  16 03:59 main_section_ldflags
-rw-r--r--. 1 root root  541 3月  15 16:32 makefile
```

以上大小可以看出，如果想要减小可执行文件大小，必须将CFLAGS和LDFLAGS同时加上。否则：

1. 单加CFLAGS，与不加没区别；
2. 单加LDFLAGS，生成的文件比单加CFLAGS小一点；
3. CFLAGS和LDFLAGS同时加上，可执行文件最小。

### readelf -a查看可执行文件

```bash
[root@localhost 06_section]# readelf -a main_normal | grep fun_
    55: 0000000000400539    39 FUNC    GLOBAL DEFAULT   13 fun_3
    61: 0000000000400512    39 FUNC    GLOBAL DEFAULT   13 fun_2
    67: 00000000004004eb    39 FUNC    GLOBAL DEFAULT   13 fun_1
    69: 00000000004004c4    39 FUNC    GLOBAL DEFAULT   13 fun_0

[root@localhost 06_section]# readelf -a main_section | grep fun_
    50: 00000000004004c9    39 FUNC    GLOBAL DEFAULT   12 fun_3
    58: 00000000004004a2    39 FUNC    GLOBAL DEFAULT   12 fun_0
```

## 参考链接

[gcc -ffunction-sections -fdata-sections -Wl,–gc-sections 参数详解](https://blog.csdn.net/pengfei240/article/details/55228228)

