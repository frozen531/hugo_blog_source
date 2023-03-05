---
title: "第13章 BigThree和移动"
date: 2023-03-05T21:21:01+08:00
draft: true
tags: ["C++Primer"]
categories: ["读书笔记"]
---


## 1. 析构函数
析构函数的 **特点**：
- 没有返回值
- 没有参数，因此不可以重载

析构函数的 **职责** 是：
1. 函数体内，释放对象使用的资源
2. 函数体执行后，销毁对象的非static数据成员，逆序释放（隐式）

<font color = "blue"><strong>隐式的销毁的对象的数据成员中，如果含有内置类型指针，不会销毁其所指向的对象</strong></font>，所以要用delete

构造函数的 **职责** 是：
1. 函数体执行前，初始化对象的非static数据成员，按类中出想的顺序顺次初始化
2. 函数体内，可以做些其他工作

在析构函数的函数体内可以执行其他操作，类对象销毁在析构函数

## 2. 拷贝、移动与交换 
用一个对象为另一个对象赋值或初始化的操作，分为 **拷贝** 和 **移动** 两种，区别在于：
- 拷贝所得的两个对象间，彼此 **互不影响**
- 移动则是将右值的对象值 **转嫁** 给另一个对象，然后这个右值可以销毁。

### 2.1 三/五原则
我们通常将：拷贝构造函数、拷贝赋值函数、析构函数看成是一个 **整体** (如果一个有需要自己定义，其他也应该一同定义)，统称其为 **拷贝控制成员** 。需要我们自己定义拷贝控制成员的 **基本原则** 是：
- 原则一：先确定该类是否需要一个析构函数，需要的话，一定要同时自定义拷贝构造和拷贝赋值函数。
- 原则二：如果需要自定义一个拷贝构造函数，则一定同时需要自定义拷贝赋值函数，反之亦然

<font color = "blue"><strong>注意：无论需要拷贝构造还是拷贝赋值，都不意味着必然需要析构函数。</strong></font>

何时需要析构函数呢？当一个类为 **管理资源** 的类时需要析构函数，即带有指针，要动态分配内存。

加入移动构造函数和移动赋值函数，提升为五原则。

### 🎃 2.2 拷贝

#### 2.1.1 拷贝构造函数
拷贝构造函数特点：
- 第一个参数必须是一个引用类型，且任何额外参数都有默认值。

```cpp
Foo (const Foo&);
```

<font color = "blue"><strong>原因是：如果传值的话，参数的传递是一个拷贝过程，那么在还没有定义完拷贝构造函数的前提下，进行类类型的拷贝，会调用该类的拷贝构造函数，如此以往。</strong></font>

>1. 直接初始化，通过函数参数匹配来选择使用最匹配的构造函数，包括：容器的emplace操作

```cpp
string dots(10, '.'); // 使用最佳参数匹配的构造函数直接初始化
string s(dots);       // 使用拷贝构造函数直接初始化
```

>2. 拷贝初始化，将右侧运算对象拷贝到正在创建的对象中，表现形式有： =，{}， 非引用类型的参数传入，非引用类型的函数返回，容器的insert和push操作

```cpp
string s2 = dots;                // 拷贝初始化，仍然使用拷贝构造函数
string null_book = "999-999-999" // 拷贝初始化，先在右侧构造一个string对象在拷贝初始化左侧，注意这里使用了构造函数的隐式转换
string nines = string(100, '9')  // 右侧直接初始化一个string对象，再拷贝初始化左侧对象
```

<font color = "blue"><strong>explicit构造函数只能用于直接初始化，所以隐式转换在拷贝初始化会受到限制。</strong></font>

#### 2.1.2 拷贝赋值函数
通过 **重载operator=运算符** 来完成。特点如下：
- 一定是成员函数，其左侧运算对象绑定到隐式的this参数
- 返回值为左侧运算对象的引用

#### 2.1.3 拷贝语义
有两种拷贝语义：
- 使类的行为像一个值。拷贝时，副本与原对象完全独立。如string类

``` cpp
class HasPtr
{
public:
    HasPtr(const std::string &s = std::string()):ps(new std::string(s)),i(0){} // 构造函数
    HasPtr(const HasPtr &p):ps(new std::string(*p.ps)),i(p.i){} // 拷贝构造函数
    HasPtr& operator=(const HasPtr &); 拷贝赋值函数
    ~HasPtr() {delete ps;} // 析构函数
private:
    std::string *ps;
    int i;
};

HasPtr& HasPtr::operator=(const HasPtr &rhs) // 使用引用，区别于使用swap
{
    auto newp = new string(*rhs.ps); // 拷贝底层string
    delete ps; // 释放旧内存
    ps = newp; // 赋值
    i = rhs.i; // 赋值
    return *this; // 返回本对象
}
```

注意：<br>
<font color = "blue"><strong>拷贝赋值函数要处理好自赋值操作，所以最安全的就是先拷贝右侧对象到一个局部临时对象，再销毁左侧对象的原有内存。</strong></font>

- 使类的行为像一个指针。共享状态，副本与原对象使用相同的底层数据。如shared_ptr类

使用shared_ptr或使用引用计数，使用引用技术如下：

``` cpp
class HasPtr
{
public:
    HasPtr(const std::string &s = std::string()):ps(new std::string(s)),i(0)， use(new std::size_t(1)){} // 构造函数，引用计数初始化为1
    HasPtr(const HasPtr &p):ps(p.ps),i(p.i),use(p.use){ ++*use;} // 拷贝构造函数，引用计数+1
    HasPtr& operator=(const HasPtr &); 拷贝赋值函数
    ~HasPtr() // 析构函数
private:
    std::string *ps;
    int i;
    std::size_t *use; // 使用动态内存来在多个对象间共享引用计数的数据
};

HasPtr::~HasPtr()
{
    if(--*use == 0)
    {
        delete ps;
        delete use;
    }
}

HasPtr& HasPtr::operator=(const HasPtr &rhs)
{
    ++*rhs.use; // 先递增右侧计数，防止自赋值出错
    
    if(--*use == 0) // 左侧递减，如果为0，释放内存
    {
        delete ps;
        delete use;
    }

    ps = rhs.ps; // 赋值
    i = rhs.i; // 赋值
    use = rhs.use;
    return *this; // 返回本对象
}
```

### 🎃 2.3 交换
管理资源的类除了需要定义拷贝控制成员，通常还要定义swap函数，这是一种很重要的优化手段。

一般交换两个对象的值，如HasPtr类对象，要使用一个中间临时对象。如果是采用类值方式，则会将原v1中的string进行2次拷贝操作，这些内存分配是不必要的，我们更希望通过交换指针来完成。

``` cpp
class HasPtr
{
    friend void swap(HasPtr&, HasPtr&);
};

inline void swap(HasPtr& lhs, HasPtr& rhs)
{
    using std::swap; // 注意点
    swap(lhs.ps, rhs.ps);
    swap(lhs.i, rhs.i);
}
```

注意：<br>
- 对于内置类型是没有特定版本的swap，所以使用标准库的swap
- 如果类内成员有自定义的swap，则会使用类型特定的swap，所以不能指定成std::swap，这样不能最优匹配。

#### 2.3.1 拷贝并交换

```cpp
HasPtr& HasPtr::operator=(HasPtr rhs) // 使用值传递
{
    swap(*this, rhs);
    return *this;
}
```

注意：<br>
- 参数值传递
- 拷贝并交换是异常安全的，可以处理自赋值问题
- 由于原来的方式是通过一个局部变量暂存，需要分配一次内存；这里使用swap，在实参传递时分配一次内存，两者这样来说是一样的


### 🎃 2.4 移动
新特性：移动而非拷贝，出现原因：
1. 很多局部或临时对象在拷贝完后就被销毁了
2. 有些类不能够拷贝，如IO类或unique_ptr类

#### 2.4.1 左值引用和右值引用
**左值持久；右值要么是字面常量，要么是临时对象**。

本质：<font color = "blue"><strong>引用是某个对象的别名。</strong></font>

- 左值引用&
> 可以绑定到：变量、返回左值引用的函数(赋值、下标、解引用、前置递增/递减运算符)
> 因为变量是持久的，知道离开作用域才被销毁，所以是左值

- 右值引用&&
> 可以绑定到返回非引用类型的函数(算术、关系、位以及后置递增/递减运算符)

特殊：<br>
1. 对于返回非引用类型的函数，除了可以绑定到右值引用，还可以绑定到const的左值引用
2. 不能将一个右值引用绑定到一个右值引用类型的变量上，如下rr3不能绑定到rr2

```cpp
int i = 42;
int && rr1 = 42;        // 正确
int &&rr2 = i * 2;      // 正确
const int &r2 = i * 2;  // 正确

int &&rr3 = rr2;        // 错误，rr2是变量，是左值
```

- std::move()

可以显式的将一个左值转换为对应的额右值引用类型

```cpp
#include <utility>
int &&rr4 = std::move(rr1); // 必须使用std::move，防止发生名字冲突
```

注意：<br>

1. 我们可以销毁一个移后源对象，可以赋予新值，但是不能够使用移后源对象的值，更不能妄加揣测值是什么
2. 必须使用std::move，防止发生名字冲突


#### 2.4.2 移动构造函数
- 移动构造函数必须确保销毁移后源对象是无害的，即要<font color = "blue"><strong>切断移后源对象与内存间的联系，使其处于可析构状态。</strong></font>
- 函数要声明为noexcept

```cpp
StrVec::StrVec(StrVec &&s) noexcept : elements(s.elements), first_free(s.first_free), cap(s.cap)
{
    s.elements = s.first_free = s.cap = nullptr; // 切断与原内存的联系，使其处于可析构状态
}
```

```cpp
HasPtr::HasPtr(HasPtr &&p) noexcept : ps(p.ps), i(p.i)
{
    p.ps = 0;
}
```


#### 2.4.3 移动赋值函数

- 移动赋值函数也要对自赋值进行判断，因为传入的值可能是std::move强制转换的左值。
- 函数要声明为noexcept

```cpp
StrVec& StrVec::operator=(StrVec &&rhs) noexcept
{
    if(this != &rhs) // 自赋值判断，很可能rhs是std::move强制转换的左值
    {
        free();
        elements = rhs.elements;
        first_free = rhs.first_free;
        cap = rhs.cap;
        
        rhs.elements = rhs.first_free = rhs.cap = nullptr; // 切断与原内存的联系，使其处于可析构状态
    }
    
    return *this;
}
```

拷贝并交换函数实际上既是拷贝赋值函数，也是移动赋值函数

```cpp
HasPtr& HasPtr::operator=(HasPtr rhs) // 使用值传递
{
    swap(*this, rhs);
    return *this;
}
```

### 🎃 2.5 函数匹配问题
#### 2.5.1 拷贝与移动

```cpp
Foo(const Foo&); // 适用于参数为任何值
Foo(Foo &&);     // 移动构造函数，只适用于参数为右值
```

- 既有拷贝构造也有移动构造
> 左值拷贝，右值移动
- 没有移动构造只有拷贝构造
> 不论左值右值，均拷贝

#### 2.5.2 右值引用和成员函数

```cpp
void push_back(X &&);       // 只适用于传入右值
void push_back(const X &);  // 适用于传入任何值
```

#### 2.5.3 引用限定符&
旧版本中可以向右值赋值

```cpp
s1 + s1 - "wow";
```

为了兼容，新版本也允许，但我们可以在自己的程序中阻止这种用法，使用 **引用限定符:&**。特点：
- 类似于const，在函数的声明和定义中都必须出现。
- 如果和const一同出现，只能在const限定符之后
- 当出现两个或以上具有相同名字和参数的成员函数时，必须对所有函数都加上引用限定符，或都不加

```cpp
// 都加
Foo sorted() &&;           // 适用于右值，即右值可调用
Foo sorted() const &;      // 适用于任何，即右值左值均可调用

// 都不加
Foo sorted(Comp*);         // 适用于右值
Foo sorted(Comp*) const;   // 适用于任何
```

## 3 合成的函数
### 🎃 3.1 拷贝控制函数
#### 3.1.1 合成

- 合成的拷贝构造函数：将对象的非static的成员逐个拷贝到另一个正在创建的对象中
- 合成的拷贝赋值函数：将对象的非static的成员逐个赋予到另一个对象的对应成员中
- 合成的析构函数执行的是：销毁对象的非static的数据成员

<font color = "blue"><strong>但合成的这些函数也可能是阻止相应操作的发生，即合成的函数可能是删除的。</strong></font>其规则如下：

- 如果一个类的数据成员不能够被默认构造、拷贝、复制或析构，则对应的成员函数将被定义为删除的
- 如果一个成员有删除的或不可访问的析构函数，则对应的析构、构造和拷贝构造函数将被定义为删除，否则会出现无法销毁的对象
- 如果一个具有引用成员或无法默认构造的const成员的类，其合成的默认构造函数是删除的
- 如果一个具有引用成员或const成员的类，则它的拷贝赋值函数是删除的

#### 3.1.2 手动定义拷贝控制函数

手动定义函数为编译器合成的版本和删除函数

```cpp
public：
    Foo() = default;
    Foo(const Foo&) = delete; // 新标准

private：
    Foo(const Foo&); // 以前，将想要删除的函数声明为私有，且只声明不定义
```

注意：
- default只能对于编译器可以合成的函数使用
- default可以在类内声明时使用，此时函数为内联函数；也可以在类外定义，此时非内联
- delete可以对任何函数使用
- delete只能出现在函数第一次声明的时候
- 声明为private访问的函数但不定义，如果用户代码使用这种函数，则在编译时出错；友元和成员函数使用，则在链接时出错。

### 🎃 3.2 移动操作
#### 3.2.1 合成

只有当类没有定义自己的任何版本的拷贝控制函数，且类中的每个非static数据成员都可以移动是，编译器才会合成它的移动构造和移动赋值运算符。

#### 3.2.2 手动默认的移动操作
使用default，如果显式声明定义了移动操作为default，而类中有不可移动的成员，则该移动操作被删除。

### 🎃 3.3 拷贝控制与移动操作的关系
#### 3.3.1 合成的函数间相互作用关系

相互作用关系：如果一个类定义了一个移动构造函数和/或移动赋值函数，则该类的合成拷贝构造函数和拷贝赋值运算符会被定义为删除的。