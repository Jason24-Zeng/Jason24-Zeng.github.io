---
title: Effective C++ (IV)
date: 2022-03-24 22:10:22
tags:
- C++
- Design & Declarations
- 书籍笔记
categories:
- coding
- C++
cover: /img/Effective-C-II/effective_cplusplus_top.jpeg
top_img: /img/Effective-C-II/effective_cplusplus_top.jpeg
---

这章主要是对于良好 C++ 接口的设计与声明提出意见与建议。而软件设计，就是以一般性的构想出发，逐步实现整体结构，以允许特殊接口开发的步骤。使用以下提到的准则，有利于我们实现代码开发的正确性，高效性，封装性，维护性，延展性以及协议的一致性。

## Item 18. 让接口容易正确使用且不易误操作

**make interfaces easy to use correctly and hard to use incorrectly**

日常工作中，程序员如果使用 C++，一定会接触到各式各样的接口。比如函数接口、类接口、模板接口。而接口是客户与程序员开发代码的互动手段。好的代码应该具有这样的要求，当客户使用某个接口时，如果该接口不能获得预期结果，则代码不该通过编译。

要实现这个要求，则程序员在设计时需要考虑客户可能产生的错误。

需要记住：

- 好的接口应该满足容易被正确使用却不容易被误用的条件。这应该是我们在实现每一个接口时需要尽可能考虑的事情。

- 接口的“正确使用”需要包括接口的一致性，以及内置类型的行为兼容。

- 接口的“不易误用”则要求在类型创立时限制类型操作，尽可能减少对象值被无意修改的可能性，消除客户的资源管理责任等。

- `shared_ptr` 允许 custom deleter，这可以防范动态链接错误，可被用来自动解除 mutex 上。

## Item 19. 像设计 type 一样设计 class

**Treat class design as type design**

与其他 Object-Orient Program Language 一样，当我们设计一个新类时，我们实际也是定义了一个新类型。而大部分时间，我们实际上都是在扩展我们的 type system。我们需要思考重载函数和操作符，控制内存的 allocate，以及定义对象初始化与终结销毁等。因此，我们需要像语言设计者设置语言内置类型一样得谨慎研究class 的设计。

对于设计每一个 class，我们都需要考虑如下的问题：

- 新类型的对象应该如何被创建和销毁？这会影响类的 constructor, desctructor, 以及内存分配与释放函数（operator new, operator new[],  operator delete 和 operator deletep[]）的设计

- 对象的初始化与对象的赋值应该有什么区别？

- 新类型的对象如果被 pass-by-balue，会发生什么？copy constructor 用来定义一个类型的 pass-by-value 如何实现

- 什么是新类型的“合法值”？

- 新类型需要配合某个 inheritance graph 继承图谱么

- 新类型需要什么样的转换

- 什么样的操作符合函数对新类型是合理的？

- 什么样的标准函数应该驳回，我们必须将其声明为 private。

- 谁该取用新类型的成员，这决定了类中哪个成员是 public 的，哪个是 protected 的，哪个是 private 的。也会帮助我们决定哪个 class 或 function 应该是 friend，以及应该实现一个怎样的嵌套关系。

- 什么是新类型的 undeclared interface

- 这个新类型有多么一般化？这个多出现在基类构建与类模板构建上。

- 真的需要新类型么？
