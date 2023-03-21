---
title: "rm删除命令"
date: 2023-03-09T21:02:38+08:00
draft: true
tags: ["命令"]
categories: [Linux]
---

## 删除文件

```bash
#递归删除文件夹下以*.o的文件
find . -name "*.o" | xargs rm -f
```

`find . -name "*.o"`在当前目录下递归寻找以`.o`结尾的文件，`xargs`将前面的搜索结果传入后面的参数。


