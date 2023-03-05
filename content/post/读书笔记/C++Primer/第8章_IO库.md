---
title: "第8章 IO库"
date: 2023-03-05T21:20:57+08:00
draft: true
tags: ["C++Primer"]
categories: ["读书笔记"]
---


## 1. IO类

```cpp```通过定义```IO```定义输入输出，包括：

| 头文件   | 类型                                                         | 说明     |
| -------- | ------------------------------------------------------------ | -------- |
| ```iostream``` | ```istream, wistream``` <br> ```ostream, wostream``` <br> ```iostream, wiostream``` | 设备读取 |
| ```fstream```  | ```ifstream, wifstream``` <br> ```ofstream, wofstream``` <br> ```fstream, wfstream``` | 文件读取 |
| ```sstream```  | ```istringstream, wistringstream``` <br> ```ostringstream, wostringstream``` <br> ```stringstream, wstringstream``` | 内存读取 |

**注意**：

- 使用继承和模板可以忽略不同类型的区别

| 基类    | 派生类                  |
| ------- | ----------------------- |
| ```istream``` | ```ifstream, istringstream``` |
| ```ostream``` | ```ofstream, ostringstream``` |

- <font color = "blue"><strong>IO对象不能拷贝，因此进行IO操作的函数以非常量的引用传递和返回</strong></font>

### 1.1 IO库的条件状态

- IO操作可能发生错误，所以IO库有条件状态：

| 条件状态      | 说明                                 |
| ------------- | ------------------------------------ |
| ```strm::badbit```  | 不可恢复的系统级错误                 |
| ```strm::failbit``` | 可恢复的错误                         |
| ```strm::eofbit```  | 文件结束，同时```failbit```置位            |
| ```strm::goodbit``` | 流未发生错误                         |
| ```strm::fail()```  | 流```s```的```failbit```或```badbit```置位，则返回```true``` |
| ```strm::good()```  | 流```s```处于有效状态，则返回```true  ```        |

```cpp
while(cin >> word) // 成功，相当于转成bool，!fail()
```

### 1.2 管理输出缓存

每个输出流都管理一个缓冲区，用于保存程序读写的数据。会导致缓冲刷新的情况如下：

> 1. 程序结束。作为```return```操作的一部分
> 2. 使用操纵符```endl```显示刷新缓冲区
> 3. 缓冲区满时，需要刷新缓冲，而后数据才能继续写入
> 4. 在每次输出操作后，用```unitbuf```设置流的内部状态，清空缓冲。```cerr```是设置```unitbuf```的，所以写到```cerr```的内容都是立即刷新
> 5. 一个输出流被关联到另一个流时，例如当读写被关联的流时，关联到的流的缓冲区会被刷新。默认情况下，```cin```和```cerr```都关联道```cout```，因此读```cin```或写```cerr```都会导致```cout```的刷新

> ```cpp
> // 通过tie()关联两个流
> cin.tie(&cerr); // 将cin关联到cerr，读取cin时会刷新cerr
> cin.tie(nullptr); // 解关联
> 
> ostream *old = cin.tie(); // 返回当前关联的流的指针，没有关联时，返回nullptr
> ```

- 操纵符

| 操纵符    | 说明                              | 说明               |
| --------- | --------------------------------- | ------------------ |
| ```endl```      | 换行并刷新缓冲区                  | ```cout << endl;```      |
| ```flush```     | 刷新缓冲区，但不输出任何字符      | ```cout << flush; ```    |
| ```ends```      | 插入一个空字符，然后刷新缓冲区    | ```cout << ends;    ```  |
| ```unitbuf```   | 设置是每次写操作后都进行```flush```操作 | ```cout << unitbuf; ```  |
| ```nounitbuf``` | 恢复正常的刷新机制                | ```cout << nounitbuf;``` |

- <font color = "blue"><strong>如果程序崩溃，则输出缓冲区是不会被刷新的</strong></font>，所以有时候输出语句实际已经执行，但由于缓冲区未被刷新，被挂起没有打印，就不会有结果显示在屏幕上
- ```string line; getline(cin, line); // 从输入流读取一整行```


## 2. 文件输入输出

- <font color = "blue"><strong>一个文件流文件需要绑定到一个文件</strong></font>

```cpp
// 1. 打开文件方法一
fstream fstrm(s); // 创建一个fstream，并打开名为s的文件

// 打开文件同时设置打开模式
fstream fstrm(s, mode); // 按指定方式打开名为s的文件

// 1. 打开文件方法二
fstream fstrm;
fstrm.open(s); // 创建一个fstream，并打开名为s的文件

// 2. 关闭与fstrm绑定的文件，返回void，关闭后可以再与其他文件绑定
fstrm.close();

// 3. 检查是否正常打开
if(fstrm) // 相当于!fail()
```

- <font color = "blue"><strong>当一个fstream对象被销毁时，close会自动调用</strong></font>。
- 文件打开模式

| 文件模式        | 说明                           |
| --------------- | ------------------------------ |
| ```in```              | 以读方式打开                   |
| ```ofstream::out ```  | 以写方式打开，同时文件内容丢弃 |
| ```ofstream::app```   | 文末追加(```append```)               |
| ```ofstream::trunc``` | 打开文件被截断                 |

## 3. string流

- <font color = "blue"><strong>一个string流需要绑定到一个string对象</strong></font>

```cpp
sstream strm; // 一个未绑定的流对象
sstream strm(s); // 绑定到一个string，保留其拷贝

strm.str(); // 返回strm锁保留的string的拷贝
strm.str(s); // 将string s拷贝到strm中，返回void
```

- 可以使用```istringstream```读取一整行的输入
- 可以使用```ostringstream```整理一整行的输出