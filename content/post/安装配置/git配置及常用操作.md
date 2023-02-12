---
title: "Git配置及常用操作"
date: 2023-02-10T20:15:58+08:00
draft: true
tags: ["git"]
categories: ["安装配置"]
---

git是一个分布式版本控制系统，用于记录版本信息。个人的电脑都拥有完整的版本控制库，即使没有网络也可以拉取分支、提交修改等，对别人没有任何影响，即所谓的分布式。当需要多人开发时，大家都拥有完整的版本库，只需要将各自修改推送给对方，就能相互看到。 --- 记录完整的文件

相比于svn(集中式版本控制系统)，所有修改都提交到一个服务器上，这样个人拉取的的分支和修改都会提交到这个服务器上，对于别人，这个修改也体现到了他的代码中，而不论这个改动是对是错。 --- 增量式记录

## 1. git基本配置
[git](https://git-scm.com/downloads)安装选择安装位置后一路next到完成。使用GitHub的用户名和注册邮箱。
```
// (username是自己的账户名)
git config --global user.name "username"   

// (useremail注册账号时用的邮箱)
git config --global user.email "useremail"     

// 查看配置
git config --global --list
```

## 2. git基本理论
本地git仓库有3部分组成：工作区、暂存区和git仓库

### 2.1 工作区
我们写和改文件的地方；

### 2.2 暂存区
临时存放改动的地方，会对加入的文件状态进行监控，并提示未加入的文件。
```
// 将工作区文件添加到暂存区
// 添加指定文件
git add [file1] [file2] ...
// 添加当前目录下全部文件
git add .
// 添加指定目录
git add [dir]

// 查看git仓库下文件状态
git status
```
```On branch master```列出当前分支，此时master分支。有三证文件状态：已修改（modified）、已暂存（staged）和已提交（committed）

### 2.3 git仓库
记录所有的版本信息。
```
// 创建git仓库
git init

// 提交暂存区内修改到git仓库
git commit -m "提交日志"
// add和commit合在一起（懒人操作）
git commit -am "提交日志"

// 查看历史记录，只能看HEAD之前的版本，但是HEAD回滚向前的话，之后的log就看不到了
git log
// 查看所有的历史记录，不被HEAD左右
git reflog
```

### 2.4 三者间的腾挪



## 3. 首次上传
```
git init
git add .
git commit -m "提交日志"
```

## 3. 删除误传文件 


## 4. .gitignore忽略指定文件


## 5. 文件重命名


## 6. 分支操作


## 参考链接
[《极客Python之Git实用教程》 ](https://fishc.com.cn/forum-334-1.html)