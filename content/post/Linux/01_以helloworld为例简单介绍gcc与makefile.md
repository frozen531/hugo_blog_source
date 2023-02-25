---
title: "01_以helloworld为例简单介绍gcc与makefile"
date: 2023-02-12T20:57:53+08:00
draft: true
tags: ["gcc","makefile"]
categories: [Linux]
---

## 1. hello.c

```c
#include <stdio.h>
// #define DEBUG

int main()
{
	printf("hello world!\n");
	#ifdef DEBUG
		printf("debug\n");
	#endif
	return 0;
}
```

## 2. gcc编译
### 2.1 编译过程
![编译与链接](编译链接过程图.png)

```gcc```帮助文档:```gcc --help```

（1）预处理（pre-processing）： \
处理条件编译等。
```c
gcc -E hello.c -o hello.i
```

（2）编译(compiling)： \
编译产生汇编文件。

```c
gcc -S hello.i -o hello.s
```


（3）汇编(assembling)： \
将汇编代码转为可执行的二进制文件

```c
// 将汇编文件转为目标文件
gcc -c hello.s -o hello.o	
//或
// 将源文件预编译、编译和汇编得到目标文件
gcc -c hello.c -o hello.o	
```

（4）链接(linking) \
处理各个模块间的相互引用关系，包括模块间函数调用和变量访问 ---> 模块间符号的引用，最终将多个单独开发编译的模块组合成一个程序。\
链接过程包括：
1. 地址和空间分配 
2. 符号决议/绑定 
3. 重定位：重新计算各个符号（函数、变量）地址的过程

```c
// 这里...省去了需要用到的其他库，最终得到可执行文件hello
ld ... hello.o -o hello	
```

注解：
1. 上面```-o```是```output```的意思，后面可以指定要输出的文件名
2. 如果想一步到位```gcc hello.c -o hello1```

另外，如果想用调试器执行一个可执行文件，则需要加上`-g`，如```gcc -g hello.c -o hello1```，但这样会在编译时创建符号表，关闭优化机制，所以生成的可执行文件接近不加`-g`时的两倍。

### 2.2 参数介绍
上面的`-E，-S`使用不多（`esc`简化记忆），其他介绍如下：
|选项| 说明 |
|--|--|
| -v/--version | 查看版本信息 |
| -o | 指定输出文件名 |
| -c | 在生成库文件的时候会使用到 |
| -I | 头文件搜索目录 |
| -L | 库文搜索的目录 |
| -l | 添加库文件名，注意文件名为掐头去尾后的剩余部分，如有库文件`libMytest.so`库文件，则这里只写`Mytest` |
| -D | 编译时定义宏，这个宏可以在文件中使用，在编译过程中指定，作为`#define 宏名`的替代，便于有多个文件中`#ifdef 宏名`的使用及修改 |
| -O | 指定代码优化等级，有0，1，2，3，0代表不优化，3优化等级最高 |
| -Wall | 打印代码中的警告信息 |
| -Werror | 将警告当做错误处理 |
| -g | 加入调试信息，最终生成的文件会比没有调试信息的文件大 |
| -t | 显示.a中打包的文件 |

假设`work`文件夹下有文件`hello.c`和头文件所在目录`inc`。`hello.c`中包含了位于`inc`文件夹下的头文件，并且使用宏控制打印信息，则可以综合运用如下：

```bash
gcc hello.c -o hello -I./inc -Wall -DDEBUG -O3 -g
```

## 3. Makefile
### 3.1 基本格式
将`2.1`节中的```gcc```过程写入到`Makefile`文件中。

```bash
// 基本格式:
// 目标：依赖
// 		命令

A: B
(tab)<command>
(tab)<command>
```
- 这里的```A```是目标名，并不需要与最后生成的文件名相一致，写成相同的话可以比较清楚该目标格式中最终生成的是那个文件。
- 执行```make```命令会打印命令```command```并执行，随后就可以发现生成相关文件啦。
- 这里的`(tab)`一定要是`tab`，不能是空格，否则会有如下报错

```bash
[root@bogon 01_hello_world]# make
makefile:2: *** 遗漏分隔符 。 停止。
```

### 3.2 编译运行
```bash
// 例如
main: hello.c
        gcc -E hello.c -o hello2.i
        gcc -S hello2.i -o hello2.s
        gcc -c hello2.s -o hello2.o
        gcc hello2.o -o hello2
```
上述可以整合为一句：
```bash
main: hello.c
        gcc hello.c -o hello2
```
执行如下：

```bash
[root@localhost 01_hello_world]# make
gcc hello.c -o hello2
[root@localhost 01_hello_world]# ./hello2
hello world!
```
- 使用`-D`控制执行流程
```bash
main: hello.c
        gcc hello.c -o hello2 -DDEBUG
```
执行如下：

```bash
[root@bogon 01_hello_world]# ./hello2
hello world!
debug
```
