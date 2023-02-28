---
title: "Shell变量及运算符"
date: 2023-02-28T22:16:27+08:00
draft: true
tags: ["Shell"]
categories: ["Linux"]
---

## Bash的变量
注意点：
1. 在Bash中，变量默认都是字符串类型。如果要进行数值运算，必须指定变量类型为数值型。
2. 变量由`=`连接，两边不能有空格。
```bash
# 变量定义
[root@bogon ~]# name=alex

# 变量调用，使用$
[root@bogon ~]# echo $name
alex

# 查看所以变量，包括用户自定义、环境变量等
[root@bogon ~]# set

# 删除变量
[root@bogon ~]# unset name
```
3. 变量的值如果有空格，需要用单引号`''`或双引号`""`；可以使用转义符`\`;
4. 变量可以叠加，类似字符串拼接。使用双引号包含`"$变量名"`或用`${变量名}`包含
```bash
[root@bogon ~]# a=aaa

[root@bogon ~]# b="$a"bbb
[root@bogon ~]# echo $b
aaabbb

[root@bogon ~]# d=dd${b}
[root@bogon ~]# echo $d
ddaaabbb
```
5. 变量值可以是命令的执行结果，命令需要用反引号或$()包含。参考基础中的特殊符号。

6. 环境变量名建议大写，便于区分。
### 用户自定义变量
本地变量，只在当前shell中生效。
### 环境变量
环境变量可以在当前shell和这个shell的子shell中生效；\
如果将环境变量写入到配置文件中，会在所有shell中生效。
```bash
# 设置环境变量，允许用户自定义
export 变量名=变量值

# 专门查看环境变量
env

# 删除变量
unset

# 查看shell树
[root@bogon ~]# pstree
init─┬─abrt-dump-oops
     ...
     ├─rsyslogd───3*[{rsyslogd}]
     #当前bash的父shell是sshd远程工具，在bash中创建了子shellpstree
     ├─sshd───sshd───bash───pstree 
     └─udevd───2*[udevd]
```
常见系统环境变量
1. `PATH`，里面存放有系统命令的执行路径，命令其实是二进制可执行文件，同样，我们可以将自己写的可执行程序的路径放入其中，这样执行的时候不需要绝对/相对路径就可直接执行。
```bash
# 查看PATH
[root@bogon ~]# echo $PATH
/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin

# 将路径添加到PATH，使用的是字符串的拼接
[root@bogon ~]# PATH=${PATH}:/00_test/01_hello_world/
[root@bogon ~]# echo $PATH
/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:
/root/bin:/00_test/01_hello_world/
[root@bogon ~]# hello1
hello world!
```
2. `PS1`，定义系统提示符的变量。它是环境变量的一种，但是`env`查看不了，`set`可以。
![shell基础_PS1](shell基础_PS1.bmp)
```bash
# 查看
[root@bogon 01_hello_world]$ echo $PS1
[\u@\h \W]$

# 修改，仅显示最后一级目录改为完整路径
[root@bogon 01_hello_world]$ PS1='[\u@\h \w]\$'
[root@bogon /00_test/01_hello_world]#
```

### 位置参数变量
命令行的输入参数。
![shell基础_位置参数变量](shell基础_位置参数变量.bmp)

#### `read`接收键盘输入
由于只有程序编写者知道需要输入几个参数，所有可以使用`read`提示用户输入，这样更直观。
```bash
#基本格式
read [选项] [变量名]

#案例
#!/bin/bash
read -t 30 -p "Please input your name: " name
#提示“请输入姓名”并等待30秒，把用户的输入保存入变量name中
echo "Name is $name "
read -s -t 30 -p "Please enter your age: " age
#年龄是隐私，所以我们用“-s”选项隐藏输入
echo -e "\n"
echo "Age is $age "
read -n 1 -t 30 -p "Please select your gender[M/F]: " gender
#使用“-n 1”选项只接收一个输入字符就会执行（都不用输入回车）
echo -e "\n"
echo "Sex is $gender"
```
选项 | 说明
---|---
-p |“提示信息”：在等待read输入时，输出提示信息
-t |秒数： read命令会一直等待用户输入，使用此选项可以指定等待时间
-n |字符数： read命令只接受指定的字符数，就会执行
-s |隐藏输入的数据，适用于机密信息的输入
### 预定义变量
变量名不能自定义，作用也是固定的。位置参数变量是预定义变量的一类。
![shell基础_预定义变量](shell基础_预定义变量.bmp)

`$?`体现在之前的命令顺序执行中的`&&`和`||`。 \
`$$`是当前shell的PID，`$PPID`是父shell的PID号。\
`./hello &`是将脚本放入后台执行。

## Bash的运算符
由于bash的变量默认都是字符串，所以要进行数值运算，需要专门的声明。

### 数字运算与运算符
- 方式1：declare声明变量类型
```bash
#基本格式
declare [+/-][选项] 变量名

#举例
#查看环境变量类型，这里注意，变量一定要被赋值，可以是空，否则报错
[root@localhost ~]# export aa=
[root@localhost ~]# declare -p aa
declare -x aa=""

[root@localhost ~]# export bb
[root@localhost ~]# declare -p bb
-bash: declare: bb: not found

#定义整形
[root@localhost ~]# a=3
[root@localhost ~]# b=5
[root@localhost ~]# declare -i sum=$a+$b
[root@localhost ~]# echo $sum
8
```

选项 | 说明
---|---
-| 给变量设定类型属性
+| 取消变量的类型属性
-i| 将变量声明为整数型（integer）
-x| 将变量声明为环境变量
-p| 显示指定变量的被声明的类型

- 方式2：expr或let数值运算工具

```bash
# 注意：+号左右必须有空格，$()里是命令执行结果，let用法一样
[root@localhost ~]# c=$(expr $a + $b)
[root@localhost ~]# echo $c
8
```

- 方式3：`$((运算式))`或`$[运算式]`

```bash
# $(())和$[]代表数值运算
[root@localhost ~]# d=$(($a+$b))
[root@localhost ~]# echo $d
8
[root@localhost ~]# e=$[$a+$b]
[root@localhost ~]# echo $e
8
```

运算符
![shell基础_数值运算符](shell基础_数值运算符.bmp)

```bash
[root@localhost ~]# aa=$(((11+3)/2))
[root@localhost ~]# echo $aa
7
[root@localhost ~]# bb=$((14%3))
[root@localhost ~]# echo $bb
2
[root@localhost ~]# cc=$((1&&0))
[root@localhost ~]# echo $cc
0
[root@localhost ~]# dd=$((1||0))
[root@localhost ~]# echo $dd
1
```

### 变量测试与内容替换
用于测试y是否存在、为空，其实可以用条件语句实现同样功能。(不需要背，使用时知道查表就行)
![shell基础_变量测试与内容替换.bmp](shell基础_变量测试与内容替换.bmp)

## 学习链接
1. [史上最牛的Linux视频教程—兄弟连](https://www.bilibili.com/video/BV1mW411i7Qf?p=63&vd_source=b6daecdfe358d06b1107a6d13e19fe3f)
