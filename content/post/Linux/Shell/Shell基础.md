---
title: "Shell基础"
date: 2023-02-16T21:29:39+08:00
draft: true
tags: ["Shell"]
categories: ["Linux"]
---

## 1. Shell简介
Shell为用户提供一个命令输入界面，也是一种强大的编程语言，一种解释执行的脚本语言，在Shell中可以直接调用Linux系统命令。
同时，Shell是一个命令解析器，将用户输入的命令转换为内核可以识别的机器语言，同时内核将硬件的执行结果通过Shell转换为我们可以读懂的语言。


![shell基础_Shell作用](shell基础_Shell作用.bmp)

### 1.1 分类
Shell分两大类，B Shell（Bourne Shell）和C Shell（主要用于BSD版的Uinx系统，语法与C语言类似），两者语法不兼容，B Shell主要有sh、Bash、ksh、psh、zsh；C Shell主要有csh、tcsh。Linux中以Bash作为用户的基本Shell，与sh兼容。

### 1.2 查看系统所支持的Shell和当前Shell
查看系统中支持的Shell，`cat /etc/shells`
```
[root@bogon ~]# cat /etc/shells 
/bin/sh
/bin/bash
/sbin/nologin
/bin/tcsh
/bin/csh
```
查询当前Shell，`echo $SHELL`
```
[root@bogon ~]# echo $SHELL
/bin/bash
```
各种Shell的切换，输入相应的Shellm名即可切换，`exit`退出
```
[root@bogon ~]# sh
sh-4.1# ls
anaconda-ks.cfg  install.log  install.log.syslog
sh-4.1# exit
exit
[root@bogon ~]#
```

### 1.3 注释
注释使用`#`，首行必须加`#!/bin/bash`，标注该脚本用`/bin/bash`解释执行。

## 2. `echo`命令
```bash
# 基本格式
echo [选项] [输出内容]
```
输出内容注意：
- 如果有空格，需要加双引号`""`
- 包含有`!`，需要用单引号`''`

### 2.1 转义字符输出
，其中`-e`选项可以支持反斜线`\`(转义符)控制的字符转换
![shell基础_echo特殊内容输出](shell基础_echo特殊内容输出.bmp)

### 2.2 指定输出字符颜色



## 3. Shell脚本的执行方式
`hello.sh`文件如下：
```
#!/bin/bash
echo "hello world."
```
两种方式：
```bash
#方式一：赋予权限后以文件路径执行（绝对或相对路径）
chmod 755 hello.sh
./hello.sh

#方式二：使用shell解释器解释执行，可以不用给权限(不常用)
bash hello.sh

#方式三：source
```

## 4. 基本功能
### 历史命令`history`
所有历史命令默认存放在`~/.bash_history`下
```bash
# 基本格式
history [选项] [历史命令保存文件]
```
选项 | 说明
--- | ---
-w | 将缓冲区中的命令希尔历史命令保存文件中
-c | 清空历史命令

历史命令默认保存1000条，满则从头删除，可在`/etc/profile`文件中修改保存条数：`HISTSIZE=1000`。

调用方式
1. 使用上、下箭头调用以前的历史命令(**最常使用**)
2. 使用`!n`重复执行第n条历史命令
3. 使用`!!`重复执行上一条命令
4. 使用`!字串`重复执行最后一条以该字串开头的命令

### tab键补全
可进行文件或目录补全，通常一次键入没反应是因为有前缀一样的，再键入一次即可看到所有以该前缀开头的文件/文件名

### 命令的别名`alias`，`unalias`
```bash
# 基本格式:alias 别名='原命令'
[root@bogon ~]# alias ab='ls -al'

# 查询别名
[root@bogon ~]# alias 
alias ab='ls -al'
alias cp='cp -i'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'

# 删除别名，基本格式:unalias 别名
[root@bogon ~]# unalias ab
[root@bogon ~]# alias 
alias cp='cp -i'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```

命令执行顺序：
1. 用绝对路径或相对路径执行的命令
2. 别名，alias仅在当前登录生效，若要永久生效，写入`~/.bashrc`
3. bash的内部命令
4. `$PATH`中环境变量定义的路径，顺序查找到的第一个

### Bash常用快捷键
![shell基础_bash常用快捷键](shell基础_bash常用快捷键.bmp)

### 输入输出重定向
- 标准输入输出
![shell基础_标准输入输出](shell基础_标准输入输出.bmp)

- 输出重定向`>`，将原本命令输出到显示器的结果保存到文件中，便于查看。
![shell基础_输出重定向](shell基础_输出重定向.png)
```bash
# 错误输出是2，需要标明
[root@bogon ~]# ab
bash: ab: command not found
[root@bogon ~]# ab 2>> err.log
[root@bogon ~]# cat err.log 
bash: ab: command not found

# 丢弃输出，/dev/null类似垃圾箱
ls &>/dev/null
```

- 输入重定向，将原本键盘输入的改为文件输入，使用不多

### 多命令顺序执行`;` `&&` `||`
![shell基础_多命令顺序执行](shell基础_多命令顺序执行.bmp)
```bash
# ;连接多条命令，各个命令间没有逻辑关系
[root@bogon ~]# ls ; date ; cd /user ; pwd
anaconda-ks.cfg  err.log  install.log  install.log.syslog
2023年 02月 18日 星期六 16:46:35 CST
-bash: cd: /user: 没有那个文件或目录
/root

# && 逻辑与 前面命令正确执行才能执行后面的命令
[root@bogon ~]# ls && echo yes
anaconda-ks.cfg  err.log  install.log  install.log.syslog
yes
[root@bogon ~]# lsa && echo yes
-bash: lsa: command not found

# || 逻辑或 前面命令错误执行才会执行后面命令
[root@bogon ~]# ls || echo yes
anaconda-ks.cfg  err.log  install.log  install.log.syslog
[root@bogon ~]# lsa || echo yes
-bash: lsa: command not found
yes

# && 与 || 一起使用判断命令是否正常执行
[root@bogon ~]# ls && echo yes || echo no
anaconda-ks.cfg  err.log  install.log  install.log.syslog
yes
[root@bogon ~]# lsa && echo yes || echo no
-bash: lsa: command not found
no
[root@bogon ~]# ls || echo yes && echo no
anaconda-ks.cfg  err.log  install.log  install.log.syslog
no
[root@bogon ~]# lsa || echo yes && echo no
-bash: lsa: command not found
yes
no
```
通过上例可以看出，`&&`与`||`共用时，并没有所谓的优先级一说，是顺序判断
- `lsa && echo yes || echo no`，lsa错误执行所以`echo yes`不执行，因为`echo yes`不执行，所以执行后面的`echo no`
- `lsa || echo yes && echo no`，lsa错误执行所以`echo yes`执行，因为`echo yes`执行，所以执行后面的`echo no`也执行

### 管道符`|`
```bash
# 基本格式：命令1的输出作为命令2的输入
命令1 | 命令2
```
注意：要求命令1必须正常执行

### grep
```bash
# 基本格式
grep [选项] "搜索内容"
```
选项 | 说明
---|---
-i| 忽略大小写
-n| 输出行号
-v| 反向查找
--color=auto |搜索出的关键字用颜色显示

### 通配符与特殊符号
- 通配符用来匹配文件名
![shell基础_通配符](shell基础_通配符.bmp)
- 特色符号
![shell基础_其他特殊符号](shell基础_其他特殊符号.bmp)
```bash
[root@bogon ~]# name=sa
[root@bogon ~]# echo "$name"
sa
[root@bogon ~]# echo '$name'
$name
[root@bogon ~]# echo $(date)
2023年 02月 18日 星期六 17:23:43 CST
```

## 5. Bash的变量
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


## 环境变量配置文件
环境变量配置主要是定义系统的默认变量，如`PATH`、`HIETSIZE`、`PS1`、`HOSTNAME`等
### source命令
修改配置文件后需要重新登录才能生效，使用source可以直接生效。
```bash
#基本格式
source 配置文件
#等价于
.配置文件
```

### 环境变量配置文件
- `/etc/profile`、`/etc/profile.d/*.sh`、`/etc/bashrc`对所有用户生效
- `~/.bash_profile`和`~/.bashrc`对当前用户生效

文件配置优先级如下：

![shell基础_环境变量文件配置优先级.bmp](shell基础_环境变量文件配置优先级.bmp)

### 其他配置文件

文件 | 说明
---|---
~/.bash_logout | 注销时生效的环境变量配置文件
~/bash_history | 历史信息配置文件
/etc/issue | 本地终端登录信息
/etc/issue.net | 远程终端登录信息
/etc/motd | 本地和远程登录，登录后的显示信息

## 学习链接
[史上最牛的Linux视频教程—兄弟连](https://www.bilibili.com/video/BV1mW411i7Qf?p=63&vd_source=b6daecdfe358d06b1107a6d13e19fe3f)
