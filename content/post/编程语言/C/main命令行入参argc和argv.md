---
title: "Main命令行入参argc和argv"
date: 2023-02-12T21:46:58+08:00
draft: true
tags: ["C"]
categories: ["编程语言"]
---

```int main(int argc, char* argv[])```对于命令执行是的输入参数，写如下代码验证学习。

```c
#include <stdio.h>

#define VALUE_BETWEEN(x,min,max) (((x)>=(min)) && ((x) < (max)))
#define MAX_NUM                 1

// 入参描述
static void TOOL_AUDIO_Usage(void)
{
	printf("\n*************************************************\n"
			"Usage: ./app <id> [name] [size] [path]\n"
			"1)id: device id.\n"
			"2)name: file name for saving.\n"
			"default:default\n"
			"3)size: file size(KB).\n"
			"default:1024\n"
			"4)path: path for saving(NULL means current path).\n"
			"default: \n"
			"\n*************************************************\n");
}

int main(int argc, char *argv[])
{
	printf("test... argc = %d, argv[0] %s \n", argc, argv[0]);
	
	// 1.参数不够
	if (argc < 2)
    {
		TOOL_AUDIO_Usage();
    	return -1;
    }
	
	// 2.第一个参数为"-h"，显示入参信息
	if (!strncmp(argv[1], "-h", 2))
	{
		TOOL_AUDIO_Usage();
    	return 0;
	}
	else
	{
		int id = atoi(argv[1]);
	    if (!VALUE_BETWEEN(id, 0, MAX_NUM))
	    {
    		printf("id must be [0,%d)!!!!\n\n", MAX_NUM);
	    	return -1;
	    }
	}    
}
```
编译后，运行：

```bash
$ ./app
test... argc = 1, argv[0] ./app 

*************************************************
Usage: ./app <id> [name] [size] [path]
1)id: device id.
2)name: file name for saving.
default:default
3)size: file size(KB).
default:1024
4)path: path for saving(NULL means current path).
default: 

*************************************************

$ ./app -h
test... argc = 2, argv[0] ./app 

*************************************************
Usage: ./app <id> [name] [size] [path]
1)id: device id.
2)name: file name for saving.
default:default
3)size: file size(KB).
default:1024
4)path: path for saving(NULL means current path).
default: 

*************************************************
```

- `argc`至少为`1`，`argv[0]`为`./app`
- `argv[i]`为字符串输入，值的判断需要用`strncmp`