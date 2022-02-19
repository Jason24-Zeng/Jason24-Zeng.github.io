---
title: Effective C++ (III)
date: 2022-01-24 00:00:35
tags:
- C++
- Resource Management
- 书籍笔记
categories:
- coding
- C++
cover: /img/Effective-C-II/effective_cplusplus_top.jpeg
top_img: /img/Effective-C-II/effective_cplusplus_top.jpeg
---

资源是一种有借有还的“东西”，一旦使用它，就需要未来的某个时间段还给系统。而资源的形式多种多样，C++ 程序中最长使用的资源就是动态分配内存 (如果不归还，会导致内存泄漏)。除此以外，file descriptors，mutex locks，图形界面的字型和笔刷、数据库连接，网络 sokcet等，都是一种资源。

资源管理的手段还不充分，但是确保资源归还给系统很重要。而这篇博客中就主要讲解一些基于对象的资源管理办法，并加入一些专属条款弥补一般化条款的不足，从而保证管理内存的对象能适当并正确的进行。

### Item 13. 使用对象管理资源

**use objects to manage resources**

首先，我们将资源放进对象内，就可以依赖 C++ 的析构函数自动调用机制确保资源被释放。

对于我们创造的一个对象（比如调用工厂函数动态分配一个对象），我们可能因为跳过释放对象语句过早退出而形成了内存泄漏。这种过早退出的可能有：

- 一个过早的 `return/continue/goto` 语句

- 抛出异常

无论是那种情况，都不是单纯通过判断 "总是执行 delete 语句" 能够解决的。因为代码可能被增加，但是人们可能无法注意到后面的 delete 语句。

许多资源被动态分配到 heap 后，被用在一个单一区域或函数内，这些资源应该在控制流离开区域时就被释放。标准库中提供的智能指针 auto_ptr 正是针对这种形式设计的。它是一种 "pointer-like object" 类指针现象。其**析构函数自动对其所指对象调用 delete**。比如下面的例子，我们调用了 `auto_ptr` 创造对象后就不用再自己释放对象对应资源了。

```cpp
void f()
{
    std::auto_ptr<Investment> pInv(createInvestment());
    ...
}

// 其中
class Investment{...};
Investment* createInvestment(); // 通过共产函数供应某特定的 Investment 对象
```

这显示了本节的两个关键想法：

1. 获取到资源后立即放进管理对象中。RAII: Resource Acquistion Is Initialization

2. 管理对象运用析构函数确保资源的释放。

注意，一定不要让多个 `auto_ptr` 指向同一个对象，因为每个 `auto_ptr` 被销毁时都会自动删除它所指的对象。为了避免这个问题，如果通过 copy constructor 或者 copy assignment 去复制`auto_ptr`，被复制的 `auto_ptr` 会变成 null，而复制后的 `auto_ptr` 拥有取得资源的唯一权。比如下例：

```cpp
std::auto_ptr<Investment> pInv1(createInvestment());  // pInv1 is not null

std::auto_ptr<Investment> pInv2(pInv1); // pInv1 is null, while pInv2 is not null

pInv1 = pInv2;  // pInv1 is not null, while pInv2 is null
```

因为 `auto_ptr` 的这个性质，使其无法成为管理动态分配资源的好工具。如果 STL 容器要求能有正常的复制行为，则不能使用 `auto_ptr`。

其替代方法是大家都应该比较了解的「引用计数型智能指针」(reference-counting smart pointer; RCSP)。它表示的是，该指针会追踪有多少对象指向了该资源，如果 counting 为 0 则会自动删除该资源。

这种方法依然会有一定的问题，主要问题出现在 cycle of references 环状引用上：即两个对象彼此互指，及时已经没有使用这两个对象了，依然因为 counting  不为 0 而不被销毁。

`shared_ptr` 就是这样一个 RCSP。

### Item 14. 资源管理类中的 copying 行为需要慎重考虑

**Think carefully about copying behavior in resource-managing classes**

上一节描述了 RAII 在资源管理中的重要意义，同时描述了智能指针如何将这些观念运用到 heap-based 的资源上。但需要注意，不是所有资源都是 heap-based，而智能指针不适合作为 resource handlers。我们对于这种情况，可能需要建立自己的资源管理类。

这一章节需要记住：

- 在 Copy RAII 对象时必须一并复制它管理的资源，因此资源的 copying 行为会决定 RAII 对象的 copying 行为。

- 普遍常见的 RAII 类拷贝行为有：阻止 copying，使用引用计数法。也可能有其他实现行为。

### Item 15. 资源管理类中提供原始资源访问

**provide access to raw resources in resource-managing classes**

我们要善用资源管理类来处理和资源之间的所有互动，而非直接处理原始资源。但是许多 APIs 都直接引用资源，这使我们只能直接绕过资源管理对象直接访问原始资源。

在实际操作中，如果我们需要一个函数可将 RAII class 对象转换成其所内含的原始对象，有两种方式可以达成目标：显式 casting 或者 隐式 casting。是否该提供一个显式转换函数（比如 get 成员函数）还是提供隐式转换，主要取决于 RAII 被设计执行的特定工作，以及它被使用的情况。最佳的设计是需要坚持 Item 18 的忠告：让接口容易被正确使用，不易被误用。做开始时需要做的就是隐藏客户不需要看的部分，但准备好客户需要的所有东西。

这一章节，需要记住：

- APIs 通常要求访问原始资源，所以每个 RAII class 都应该提供一个 ”取得所管理资源“ 的方法。

- 对原始资源的访问可以有显式转换，也可以是隐式转换。通常，显式转换更加安全，但隐式转换对客户更加方便。

#### Item 16. 使用相应的 new 和 delete 操作时要采用相同的形式

一开始，文中举了个例子：

```cpp
std::string* stringArray = new std::string[100];
...
delete stringArray;
```

看起来没毛病，但 `stringArray` 中所含的 100 个 `string` 对象中 99 个不太可能被适当得删除，因为它们的析构函数很可能没被调用。

当使用 `new` 时，有两件事发生

1. 内存通过 `operator new` 的函数被分配出来

2. 针对这个内存，一个（或多个）构造函数被调用。

而当使用 `delete` 时，也有两件事发生

1. 针对这个内存会有一个（或多个）构造函数被调用

2. 内存通过 `operator delete` 的函数被释放

上面的问题在于，被删除的内存之内存在多少对象？这个问题决定了多少个析构函数必须被调用。数组所用的内存通常还包括 ”数组大小“ 的记录，从而方便 `delete` 调用析构函数的次数。

当我们对一个指针使用 `delete`，唯一能让其知道 ”数组大小的方式“ 就是显式得告诉它。如果 `delete` 时加上方括号，则表明指针指向的是数组，否则认为其指向单一对象。因此我们需要如下得匹配删除相应的对象

```cpp
std::string* stringPtr1 = new std::string;
std::string* stringPtr2 = new std::string[100];
...
delete stringPtr1;
delete [] stringPtr2;  // 删除一个由对象组成的数组
```

当程序员以 `new` 创建 `typedef` 类型对象时， `typedef` 的作者必须说清该以哪种 `delete` 形式删除它。而为了避免 `typedef` 数组类型带来的不清楚使用哪个 `delete` 函数的问题，我们最好不要对数组形式做 `typedef` 动作。

### Item 17. 用独立语句将新建 (newed) 对象存储进智能指针

**Storing newd objects in smart pointers in standalone statements**

不要写如下的函数：

```cpp
processWidget(std::shared_ptr<Widget>(new Widget), priority())
```

因为这个函数在执行之前需要执行三个步骤:

1. `new Widget`

2. `priority()`

3. `call shared_ptr function`

其中 `priority()` 这个函数的调用次序不固定。我们假定函数当前遵循上述调用步骤，如果`priority()` 抛出异常，我们就碰到了 Item 13 中谈到的问题，`new Widget` 对象因为没有使用 `shared_ptr` 函数调用而无法自行销毁，这样便容易造成内存泄漏。

正确的做法是，使用独立语句新建对象，然后将智能指针传给函数：

```cpp
std::shared_ptr<Widget>  pw(new Widget);

processWidget(pw, priority());
```

这样，我们就保证了执行秩序，从而避免了一些内存泄漏问题的发生，或者说避免了分开 `new Widget` 与 `call shared_ptr function` 的动作。
