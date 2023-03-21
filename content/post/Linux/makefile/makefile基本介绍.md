---
title: "Makefile基本介绍"
date: 2023-02-12T21:26:08+08:00
draft: true
tags: ["makefile"]
categories: ["Linux"]
---

## 1. 工具简介
在工程文件中，按照类型、功能、模块等将源文件分布于不同的目录下，```makefile```的好处是描述Linux下一个工程的编译和链接规则，使工程编译自动化执行。


## 2. 编写格式和规范
### 2.1 文件命名规则

makefile文件的命名通常为`makefile`和`Makefile`，但也可以是其他任意名字，但要通过`-f`执行。

```bash
[root@bogon 05_static]# make -f makefile_path
[root@bogon 05_static]# make clean -f makefile_path
```

### 2.2 基本格式
```c
目标:依赖
	命令

// 如：
A: B C...
(tab)<command>
(tab)<command>
```

注意： 
- 每个命令行前都必须键入```tab```；
- 依赖可以有多个，空格隔开；

## 3. 执行过程

### 3.1 make执行顺序

`make`时，默认查找顺序：当前目录下依次寻找"GNUmakefile"、"makefile"和"Makefile"，并默认生成第一个目标。

- ```make```命令生成```makefile```文件中的终极目标（第一个规则）
- 在最上一个规则写我们最终要生成的目标文件（终极目标），对于其中的依赖项，如果没有相应文件，则向下查找生成该依赖文件的规则（子目标，为终极目标提供依赖）

```c
// 可以生成hello5.o和hello5
main: hello5.o
     gcc hello5.o -o hello5
hello5.o :hello.c
     gcc -c hello.c -o hello5.o

// 只能执行第一个目标格式中命令，只生成hello6.o
hello6.o :hello.c
	gcc -c hello.c -o hello6.o
main: hello6.o
	gcc hello6.o -o hello6
```

- 也可以指定目标，如：`make clean`。

- 当有文件修改后，并不是所有的文件都会被编译，只有发生改变后的文件会重新编译一次，并且终极目标规则一定会被编译
>对于目标是否需要重新编译更新，工作原理是文件生成时间的对比，以提高编译效率
>1. `.o`和`.c`的时间，如果`.c`比`.o`晚，则对`.c`进行重新编译
>2. 在生成终极目标（例如生成名为`app`的文件）时，`app`的时间比`.o`晚，则会重新生成一个`app`文件）

### 3.2 执行伪目标
可以使用伪目标让`makefile`执行不同的操作，如：定制`Debug`和`Release`版本，清空中间生成文件，安装软件包等。

```bash
main: hello7.o
	gcc hello7.o -o hello7

hello7.o :hello.c
	gcc -c hello.c -o hello7.o

.PHONY:clean
clean:
	rm hello7 hello7.o -f
```

- 由于`clean`后面没有依赖文件，所以不会顺着上面执行`clean`中的命令行
- 执行方法：```make 伪目标名```命令执行对应的伪目标，例如`make clean`
- `-f`强制执行，不论删除的文件是否存在，就是不会有文件不存在的提示了
- `.PHONY`声明`clean`为伪目标，不会与本地同名文件进行是否更新的比较


### 3.3 命令参数

`man make`查看选项。

- 通过`-f`选项指定文件，而不在使用上面名字，例如`test.make `。

```bash
[root@localhost 01_hello_world]# make -f test.make 
gcc hello.c -o hello2 -DDEBUG
[root@localhost 01_hello_world]# ./hello2 
hello world!
debug
```

- `-C`指定makefile路径

```bash
[root@localhost ~]# make -C /00_test/01_hello_world/ -f test.make
make: Entering directory `/00_test/01_hello_world'
gcc hello.c -o hello3 -DDEBUG
make: Leaving directory `/00_test/01_hello_world'
```

## 4. make执行步骤

1. 读入所有的makefile
2. 读入被include的其他文件
3. 初始化文件中的变量
4. 推到规则并分析规则
5. 为所有的目标文件创建依赖关系链
6. 根据依赖关系，决定哪些目标要重新生成
7. 执行生成命令

1~5为第一阶段，6~7为第二阶段。在1~5中变量并不会马上展开，只有变量在规则中被依赖且决定要使用，变量才会展开。


## 参考链接
1. [Makefile的使用](https://blog.csdn.net/Black_Cat_33/article/details/123184325)
2. [Makefile经典教程(掌握这些足够)](https://blog.csdn.net/ruglcc/article/details/7814546)