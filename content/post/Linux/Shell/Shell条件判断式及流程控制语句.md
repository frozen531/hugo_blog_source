---
title: "Shell条件判断式及流程控制语句"
date: 2023-02-28T22:07:31+08:00
draft: true
tags: ["Shell"]
categories: ["Linux"]
---

## 条件判断

两种执行方式：`test`和`[  ]`（常用），注意：`[]`内必须有空格

- 按照文件类型判断，常用为标蓝的

![shell基础_按照文件类型进行判断](shell基础_按照文件类型进行判断.bmp)

```bash
#使用test -e判断
[root@localhost 03_make_set]# test -e makefile && echo yes || echo no
yes

#使用[ -e file ]判断
[root@localhost 03_make_set]# [ -e makefile ] && echo yes || echo no
yes
```

- 按照文件权限判断

![shell基础_按照文件权限进行判断](shell基础_按照文件权限进行判断.bmp)

- 两个文件之间比较

![shell基础_两个文件之间进行比较](shell基础_两个文件之间进行比较.bmp)

- 两个整数之间比较

![shell基础_两个整数之间比较](shell基础_两个整数之间比较.bmp)

- 字符串比较

![shell基础_字符串的判断](shell基础_字符串的判断.bmp)

- 多重条件判断

![shell基础_多重条件判断](shell基础_多重条件判断.bmp)

## 流程控制

### if语句

```bash
#基本用法
#单分支
if [ 条件判断式 ];then
     程序
fi
#或
if [ 条件判断式 ]
     then
          程序
fi

#多分支
if [ 条件判断式 ]
     then
          程序
elif [ 条件判断式 ]
     then
          程序
else
          程序
fi

#举例
if [ -e $svn_dir ] 
then 
    #先up一下，否则出来的SVN版本号可能不对，将打印丢弃
	svn up > /dev/null       
    #svn info信息中截取版本号
	svn_version=$(svn info | grep "Last Changed Rev:" | awk -F ': ' '{print $2}')
else
	echo svn_dir is not exist
	exit 1
fi
```

注意：
1. 以`if`开头，`fi`结尾；
2. `if [ 条件语句 ]`中`if`后的空格，`[]`内的空格都不能省略；否则会有相应报错，如`[13: command not found`、`syntax error near unexpected token then`。


### case语句

case只能判断一种条件关系，而if语句可以判断多种条件关系。

```bash
#基本用法
case $变量名 in
     "值1")
          程序
          ;;
     "值2")
          程序
          ;;
     ...
     *)
          程序
          ;;
esac

#举例
#!/bin/bash

read -p "please choose yes/no:" -t 30 cho

case $cho in
        yes)
                echo "your choose is yes!"
                ;;
        no)
                echo "your choose is no!"
                ;;
        *)
                echo "your choose is error!"
                ;;
esac
```

注意：
1. `case`开头，`esac`结尾；
2. `;;`表示程序段的结束，不可省略；

### for循环

```bash
#基本格式1
for 变量 in 值1 值2 ...
     do
          程序
     done

#举例
#!/bin/bash

for i in 1 2 3 4 5
        do
                echo $i
        done

#基本格式2
for ((初始值; 循环控制条件; 变量变化))
     do
          程序
     done

#举例
#!/bin/bash

s=0;
for ((i=1; i<=100; i++))
        do
                s=$(($s+$i))
        done

echo $s
```

注意：
`do`和`done`代替`{}`

### while循环与until循环

while为不定循环、条件循环，for是固定循环。

```bash
#基本格式
while [ 条件判断 ]
     do
          程序
     done

#举例
#!/bin/bash

i=1
s=0
while [ $i -le 100 ]
        do
                s=$(($s+$i))
                i=$(($i+1))
        done

echo $s
echo $i
```

while是条件满足进行循环，until是条件不满足时进行循环。

```bash
#基本格式
until [ 条件判断式 ]
     do
          程序
     done

#举例
#!/bin/bash

i=1
s=0 
 
until [ $i -gt 100 ]
        do
                s=$(($s+$i))
                i=$(($i+1))
        done

echo $s
echo $i
```

## 学习链接
1. [史上最牛的Linux视频教程—兄弟连](https://www.bilibili.com/video/BV1mW411i7Qf?p=63&vd_source=b6daecdfe358d06b1107a6d13e19fe3f)