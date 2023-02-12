---
title: "Git常见问题"
date: 2023-02-11T14:58:36+08:00
draft: true
tags: ["git"]
categories: ["安装配置"]
---

win11下使用git，遇到如下问题做记录。

## 1. git status中文乱码显示数字
```
// 配置core.quotepath为false
git config --global core.quotepath false
```

## 2. add操作```LF will be replaced by CRLF the next time Git touches it```
主要是换行符不一致问题

Dos/Windows平台默认换行符：回车（CR）+换行（LF），即’\r\n’ \
Mac/Linux平台默认换行符：换行（LF），即’\n’ \
企业服务器一般都是Linux系统进行管理，所以会有替换换行符的需求
```
#提交检出均不转换
git config --global core.autocrlf false
```
需要注意这个修改后，检入检出都不进行换行符的转换，对于跨平台开发会有影响。

## 参考链接
1. [git status中文乱码怎么办](https://www.php.cn/tool/git/485156.html)
2. [Git: ‘LF will be replaced by CRLF the next time Git touches it‘ 问题解决与思考](https://blog.csdn.net/Babylonxun/article/details/126598477)