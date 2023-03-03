---
title: Effective C++ (II)
date: 2022-01-23 17:30:25
tags:
- C++
- 书籍笔记
- 构造
- 析构
- 赋值
categories:
- Coding Language
- C++
cover: /img/Effective-C-II/effective_cplusplus_top.jpeg
top_img: /img/Effective-C-II/effective_cplusplus_top.jpeg
---

#### 前言

一个月前，因为工作调 动原因，有了充分的时间去拜读 C++ 相关书籍，于是决心从 *effetive C++* 开始，通过简单得学习了前 8 个条款，感觉自己茅塞顿开，发现自己之前工作中的许多不愉快都源于自己的代码习惯不够好。希望自己能在入职前，将这本工程师必读之作整体拜读一次，并以博客代替日记的方式记录下来，以备未来回顾。

#### Item 09. 不要再构造函数与析构函数中调用虚函数

**Never call virtual functions during construction or destruction**

使用 C++ 时，不要在这两个函数中调用 virtual 函数，否则可能会得到预想以外的结果。这时 C++ 与 Java 或 C# 的区别。

下面有一个看似合理，实则反直观的例子，来解释这种行为的不合理性：

```cpp
// 构建一个 base class, 创建交易对象，同时调用函数记录日志
class Transaction {
  public:
    Transaction();
    virtual void logTransaction() const = 0;    // base 中的日志记录接口
};

Transaction::Transaction() {
    ...
    logTransaction();                        // base 构造函数中调用 virtual 函数
}


class BuyTransaction : public Transaction {
  public:
    virtual void logTransaction() const;    // 继承类 BuyTransaction 内定义的日记函数
    ...
};

class SellTransaction : public Transaction {
  public:
    virtual void logTransaction() const;    // 继承类 SellTransaction 内定义的日记函数
    ...                                
}
```

这上面的函数继承构造看似合理，但如果我们初始化一个 `BuyTransaction` 对象，就会发现问题了。初始化构造继承类对象时，基类对象的成分会首先构造妥当，这个行为发生在继承类单独成分构造之前。而基类 `Transaction` 构造时会调用 `logTransaction` 函数，且被调用的是 `Transaction` 中的版本，而非 `BuyTransaction` 中的版本。导致我们如上初始化的 `BuyTransaction` 对象表现与基类一样。也就这样理解，在 base class 构造期间，virtual 函数并非 virtual 函数。

而这种先完全构造好 base class 的行为也是合理的：如果基类初始化时调用了继承类的成员，而这些成员未初始化，则大大增加了不确定性。

唯一能避免上面的情况的做法就是，确定我们构造函数与析构函数都没有调用 virtual 函数，且它们调用的所有函数也服从这一约束--不调用 virtual 函数。

那上面这个问题怎么解决呢？也就是如何保证对象创建时能有正确版本的 `logTransaction` 函数被调用？

一种做法是，虽然我们没办法把使用 `virtual` 函数从基类向下调用，但是可以*让继承类向上传递足够信息给基类构造函数*

```cpp
class Transaction {
  public:
    explicit Transaction(const std::string& logInfo);
    void logTransaction(const std::string& logInfo) const; // non-virtual
    ...
};

Transaction::Transaction(const std::string& logInfo) {
    ...
    logTransaction(logInfo);
}

class BuyTransaction: public Transaction {
  public:
    BuyTransaction(parameters) : Transaction(createLogString(parameters)) 
    {...}    \\ 将 log 信息传给基类构造函数
  private:
    static std::string createLogString(parameter);
};
```

注意到`private static` 的用法。比起使用成员初值列 (member initialization list) 的基类初始化方法，利用辅助函数传递值给基类的方法更方便。且此函数为 `static`，就保证了在基类初始化时指向的继承类成员变量是已经初始化好了的。

#### Item 10. 令赋值操作符返回一个 \*this 的引用

**Have assignment operators return a reference to \*this**

在赋值时，对于所有内置类型以及 STL 库中提供的类型如 `string`, `vector`,`complex`, `tr1::shared_ptr`等，我们发现 C++   可以满足连续赋值的形式，比如 

```cpp
int x, y, z;
x = y = z = 15;
```

为了实现这样的连续赋值，赋值操作符需要返回一个指向操作符左边实参的引用，这时一个我们在为 classes 实现赋值操作时应该遵守的协议：

```cpp
class Widget {
  public:
    ...
    Widget& operator=(const Widget& rhs) {
    // 返回指向当前对象的引用
        ...
        return *this;
    }
    // 不仅标准赋值形式适用，还适用于所有赋值相关运算
    Widget& operator+=(const Widget& rhs) {
        ...
        return *this;
    }
};
```

注意，这个协议不是强制的，因此，即使不遵守，在编译时也能通过，但是因为几乎所有标准类型都遵守这个协议，所以建议在自己定义的时候也遵守。

#### Item 11. 在 `operator=` 操作符里面处理自赋值

**Handle assignment to self in operator=**

自赋值，或者自我赋值，是指对象被赋值给自己，典型的场景有如下几个：

```cpp
// 声明一个对象，赋值给自己
Widget w;
...
w = w;


// 当 i = j 时，自我赋值
a[i] = a[j];

// 当 px 与 py 指向同一个东西
*px = *py;
```

这些不明显的自赋值现象，都是 alias 别名带来的结果。一般来说，如果某段代码操作指针或者引用，而它们又被用来"指向多个想同类型的对象"时，就可能出现指向同一个对象的情况。

##### 自赋值陷阱例子

如果遵循后面要提到的 Item 13 与 Item 14，我们会运用对象来管理资源，且可以确定资源管理对象在 copy 发生时有正确的行为。此时，赋值操作符自赋值也许是安全的。但是，如果尝试自行管理资源（写一个用于资源管理的 class 时），我们就可能掉进在停止使用资源之前意外释放资源的情况。文中举了一个这样的例子，我们建立一个 class 来保存一个指针，该指针指向一块动态分配的 bitmap：

```cpp
class Bitmap {...};
class Widget {
    ...
  private:
    Bitmap* pb;
};
```

我们需要实现一个 `operator=`赋值操作符，去更新位图 

##### Version 1

最初始的版本，貌似合理，但是如果 \*this 和 rhs 是同一个对象，则可能出现问题

```cpp
Widget& Widget::operator=(const Widget& rhs) {
    delete pb;
    pb = new Bitmap(*rhs.pb); // 可能会因为自赋值而报错
    return *this;
}
```

##### Version 2

修改版本，先判断是否自赋值，再执行。这个保证了自我赋值是安全的，但是如果 `new Bitmap` 导致异常，会产生一个指向被删除 `Bitmap` 的指针，我们无法安全删除，也无法安全读取，因此这一版本的 `operator=` 不具备异常安全性。

```cpp
Widget& Widget::operator=(const Widget& rhs) {
    if (this == &rhs) return *this;
    delete pb;
    pb = new Bitmap(*rhs.pb); // 可能会因为自赋值而报错
    return *this;
};
```

##### Version 3

把焦点放在异常安全性上，因为当 `operator=` 具备异常安全性时，它自然就具备了自赋值安全性。如下

```cpp
Widget& Widget::operator=(const Widget& rhs) {
    Bitmap* pOrig = pb;
    pb = new Bitmap(*rhs.pb); // copy assignment;
    delete pOrig;
    return *this;
};
```

需要注意，自赋值的时候，依然需要复制一份原来的值，可能不高效。可以考虑使用 Version 2 的方法加入一个 identity test，但需要考察是否有必要，比如这种自赋值出现的频率是否能 cover 住新增控制流分支与代码的效率降低。

##### Version 4

采用 copy and swap 技术，这个技术在 pb 文件的赋值操作中经常用到。

```cpp
class Widget {
    ...
    void swap(Widget& rhs);     // 交换 *this 和 rhs 数据
    ...
};
Widget& Widget::operator=(const Widget& rhs) {
    Widget temp(rhs);
    swap(temp);
    return *this;
}
// or 
Widget& Widget::operator=(const Widget rhs) {
    swap(rhs);
    return *this;
}
// 损失了函数的清晰性。
```

#### Item 12. 复制对象的每一个成分

**Copy all parts of an object**

当我们不用编辑器默认生成的 copying 函数，而自己声明 copying 函数时，即使这个 copying 函数有明显的错误，比如少引入了对象的一部分 private 成员变量时，编辑器依然不会报错。所以，如果我们在 class 中添加一个成员变量，我们必须同时修改 copying 函数。我们需要修改所有构造函数，以及任何非标准形式的 `operator=`。

一旦还有继承类，这种潜在的危机就更不容易被察觉。有些继承类，其 copy 构造函数看似copy 了类里所有声明的变量，但是因为没有制定实参传给基类构造函数，所以基类成分会被不带实参的基类 default 构造函数初始化。copy assignment 操作符的整体流程同上。

在任何时候只要我们为继承类编写 copying 函数，我们就必须小心赋值其基类成分。需要注意这些成分是 private 的，所以无法直接访问它们，而应该用继承类的 copying 函数调用相应的基类函数。

综上，当我们需要编写一个 copying 函数时，我们需要保证

1. 复制所有的 local 成员变量

2. 调用所有基类适当的 copying 函数

需要注意的是，令 copy 赋值操作符调用 copy 构造函数是不合理的，这时在构造一个存在的对象。同样，让 copy 构造函数调用 copy 赋值操作符，同样无意义，我们无法对一个为构造好的对象赋值。

如果 copy 构造函数和 copy assignment 操作符有相近的代码，最好的处理方式是建立一个成员函数供两者调用，这个函数往往是 private 且常被命名为 `init`。（是不是很熟悉？）
