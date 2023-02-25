---
title: "Makefile模板"
date: 2023-02-22T18:55:54+08:00
draft: true
tags: ["makefile"]
categories: ["Linux"]
---

## 模板
常用模板可总结如下：

```bash
CC = $(CROSS_COMPILE)gcc

# 路径
WORK_DIR:=$(WORK_PATH)/work

# 头文件
INC_DIR := -I$(WORK_DIR)/inc

# 源文件
SOURCES:=$(wildcard $(WORK_DIR)/src/*.c)

# 目标文件
OBJECTS:=$(patsubst %.c, %.o, $(SOURCES))

# 编译和链接选项
CFLAGS := -O2 -Wall -Werror -DSVN_VERSION=$(SVN_V)
LDFLAGS := -lpthread -lm -lrt -ldl 

TARGET:=app
$(TARGET):$(OBJECTS)
	$(CC) $(LDFLAGS) -o $@ $^ $(LIBS) $(SHARE_LIBS)

%.o:%.c
    @echo build $(notdir $@)
	$(CC) $(INC_DIR) $(CFLAGS) -c $< -o $@

.PHONY : clean
clean:
	@-rm -f $(OBJECTS) $(TARGET)
```

## 解读
### 执行顺序

makefile从第一个目标`$(TARGET)`开始执行，该目标的依赖文件是`$(OBJECTS)`，此时还没有生成`$(OBJECTS)`，查找生成这些依赖文件的规则，然后倒序最后生成最开始也是终极的目标文件`$(TARGET)`。

### gcc选项CFLAGS和LDFLAGS

可以细分为`CFLAGS`和`LDFLAGS`，分别用于编译和链接

### 模式规则$@、$<、$^、$?、%.o

makefile除了可使用shell中的通配符，还有自己专用的，只能够在规则命令中使用。

符号 | 说明
--- | ---
`$@`|所有目标文件，即上面的$(TARGET)，%.o
`$<`|第一个依赖文件
`$^`|目标依赖的所有文件，即上面的$(OBJECTS)列表
`$?`|所有更新过的依赖文件
`%.o:%.c`|`%`表示取出来文件的文件名，是匹配符

上面特殊符号的转述：

```bash
$(TARGET):$(OBJECTS)
	$(CC) $(LDFLAGS) -o $@ $^ $(LIBS) $(SHARE_LIBS)
#$@是目标文件$(TARGET)即app
#@^是所有依赖文件$(OBJECTS)即目标文件列表
#app:a.o b.o c.o
#    $(CC) $(LDFLAGS) -o app a.o b.o c.o $(LIBS) $(SHARE_LIBS)

%.o:%.c
    @echo build : $(notdir $<) $(notdir $@)
	$(CC) $(INC_DIR) $(CFLAGS) -c $< -o $@
#%.o:%.c遍历操作，所以每次遍历中，目标文件和依赖只有其中一个
#$<第一个依赖文件，用$^也是一样的
#$@目标文件，对应的.c
#echo build的打印可以很好的看出符号对应的值是什么
```

### 命令前缀 @ 和 -

#### @

规则执行时会在屏幕上打印，如果不想打印出来，则在命令前加`@`。

```bash
%.o:%.c
	echo build $(notdir $@)
#打印
#echo build a.o
#build a.o

%.o:%.c
	@echo build $(notdir $@)
#打印
#build a.o
```

#### -

```bash
clean:
    -rm *.o
    -rm *.a
```

`-`明确表示如果rm过程中出现文件不存在等报错信息，继续执行；也可以用`rm -f`强制执行。

## 调试打印
shell命令只能用于规则中，调试打印可以使用`echo`，也可以使用makefile的`warning`。

```bash
$(TARGET) : $(OBJECTS)
	$(warning CFLAGS:$(CFLAGS))
	@echo finish:$@ $^
	@$(CC) $(INC_DIR) $(LDFLAGS) -o $@ $^ $(LIBS) $(SHARE_LIBS)
```

