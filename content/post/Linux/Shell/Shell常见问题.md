---
title: "Shell常见问题"
date: 2023-02-16T22:33:39+08:00
draft: true
tags: ["Shell"]
categories: ["Linux"]
---

## 
因为Linux和Windows使用的换行符不一样，在Linux cat -A查看全部内容
```

```
可以发现，所以提示不识别`^M`，所以需要命令`dos2unix`做格式转换。
```
dos2unix
```
