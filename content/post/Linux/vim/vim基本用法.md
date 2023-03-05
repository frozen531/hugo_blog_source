---
title: "Vim基本用法"
date: 2023-03-02T06:54:46+08:00
draft: true
tags: ["vim"]
categories: [Linux]
---

vim是一个功能强大的文本编辑器。

## 1. 模式切换

vim在3种模式间切换：

- 命令模式：常见的复制、粘贴等操作
- 插入模式：文本编辑
- 编辑模式：在命令模式下输入`:`进入，可进行搜索，设置等

![vim_工作模式](vim_工作模式.bmp)

## 2. 命令模式->插入模式

- a：append，追加，插入在光标后和行末
- i：insert，插入，光标当前处插入
- o：new a line below(o)/above(O)

![vim_插入命令](vim_插入命令.bmp)

## 3. 命令

### 3.1 定位命令

![vim_定位命令](vim_定位命令.bmp)

### 3.2 删除命令

![vim_删除命令](vim_删除命令.bmp)

### 3.3 复制和剪切命令

![vim_复制和剪切命令](vim_复制和剪切命令.bmp)

### 3.4 替换和取消命令

![vim_替换和取消命令](vim_替换和取消命令.bmp)

### 3.5 搜索和搜索替换命令

![vim_搜索和搜索替换命令](vim_搜索和搜索替换命令.bmp)

命令 | 作用
---|---
`:set icno` | 取消忽略大小写


### 3.6 保存和退出命令

![vim_保存和退出命令](vim_保存和退出命令.bmp)

### 3.7 分屏

```bash
[root@bogon 05_static]# tree
.
├── add
│   └── add.c
├── div
│   └── div.c
├── mul
│   └── mul.c
└── sub
    └── sub.c
```

命令 | 作用
---|---
:sp [文件名] | 水平分屏（split）
:vsp [文件名] | 竖直分屏（vertical split）
ctrl + w + w | 切换窗口（windows）

![vim_分屏](vim_分屏.bmp)

## 4. 永久生效的配置

vim的配置文件位置

- 普通用户：`/home/usrname/.vimrc`
- root用户：`/root/.vimrc`

![vim_vim配置](vim_vim配置.bmp)

## 学习链接
1. [史上最牛的Linux视频教程—兄弟连](https://www.bilibili.com/video/BV1mW411i7Qf?p=63&vd_source=b6daecdfe358d06b1107a6d13e19fe3f)
2. [vim中文手册](https://vimcdoc.sourceforge.net/doc/quickref.html)
3. [Vim学习笔记整理](https://zhuanlan.zhihu.com/p/129354923)