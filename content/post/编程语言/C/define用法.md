---
title: "Define用法"
date: 2023-02-12T21:40:16+08:00
draft: true
tags: ["C"]
categories: ["编程语言"]
---

下面内容来自《C和指针》
## 1. 预处理
`define`是C语言中的预处理命令，C代码在编译过程的第一步是预处理（preprocessing），这一步主要完成一些文本性质的操作：
1. 删除注释
2. 插入被`#include`指令包含的文件内容
3. 定义和替换有`#define`指令定义的符号
4. 确定代码的部分内容是否应该根据一些条件编译指令进行编译

## 2. define宏与函数间的优劣
| 属性 | define | 函数 |
|--|--|--|
| 代码长度 | 适用于非常小的代码，因为会在编译时展开 | 代码只出现在一处，其他地方调用 |
| 执行速度 | 更快 | 存在函数调用和返回的额外开销 |
| 参数类型 | 宏与类型无关，只要参数合法，可用于任何参数类型 | 函数参数与类型有关，不同的参数需要不同的函数 |

## 3. define用途
**使用格式**

```c
#define name(parameter-list) stuff
```
- `parameter-list`：为所定义的宏名
- `stuff`：以是常数、表达式、格式串等。

### 3.1 替换文本

```c
#define PI 			3.14			// 替换数值字面值常量
#define REG 		register		// 为关键字创建别名
#define DO_FOREVER 	for( ; ; )		// 同更具描述性的符号代替无限循环语句
#define CASE 		break;case	 	// 定义简短记法
```

**注意**

- `#define`不以`;`结尾
- 习惯上大写命名

### 3.2 数值表达式求值

```c
#define SQUARE(x) x * x
SQUARE(5 + 1) => 5 + 1 * 5 + 1 = 11	// 不是我们想要的结果

//=======
#define SQUARE1(x) (x) * (x)
SQUARE1(5 + 1) => (5 + 1) * (5 + 1) = 36

//=======
#define ADD(x) (x) + (x)
ADD(5 + 1) * 3=> (5 + 1) + (5 + 1) * 3 = 24	// 不是我们想要的结果

#define ADD1(x) ((x) + (x))
ADD1(5 + 1) * 3=> ((5 + 1) + (5 + 1)) * 3 = 36
```

**注意**

- 对于数值表达式求值需要加括号，保证正确运算
- 由于文本展开，运算会受到上下文环境的影响，所以要加上括号，否则会产生不可预料的计算结果，区别于函数调用。

### 3.3 类型作为参数传递

```c
#define MALLOC(n, type) \
	((type*)malloc((n) * sizeof(type)))
p = MALLOC(25, int); => p = MALLOC((int*)malloc((25) * sizeof(int)));
```

### 3.4 带副作用的宏参数

```c
#define MAX(a, b) ((a) > (b) ? (a) : (b))
int x = 5;
int y = 8;
int z = MAX(x++, y++);
printf("x = %d, y = %d, z = %d \n", x, y, z);
```
**结果**

```c
x = 6, y = 10, z = 9
```
**注意**

- 上面的传参属于带副作用的宏参数，导致预料之外的值，区别于函数调用。

### 3.5 参数传递

```c
#define PRT_INFO(arg...)    PRT_DbugPrint(DEBUG_INFO, MOD_ID,__FILENAME__,__LINE__,##arg)
PRT_INFO("a %d b %d c %d\n",a,b,c);
```

- `##` 把它两边的符号连接成一个符号

### 3.6 组合使用

```c
typedef struct
{
	UINT val1;
	UINT val2;
	void* pFunc;
}MY_STRUCT;

#define MAKE_HOST_CMD(_a,_b,_c,_d,_func_)	\
	{(_a&0xffff)|((_b&3)<<16),((_c&0xffff)<<16)|(_d&0xffff), _func_}

MY_STRUCT my_list[]=
{
	MAKE_HOST_CMD(1, 2, 3, 4, func1),
	MAKE_HOST_CMD(1, 2, 3, 4, func2),
};
```

