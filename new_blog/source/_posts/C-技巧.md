---
title: C++ 技巧
date: 2023-02-04 18:00:06
tags: 
- C++
categories:
- C++
passwd: 12346
cover: /img/SearchEnginee/searchplatform.png
top_img: /img/SearchEnginee/searchplatform.png
---

## C++ 技巧

C++ 技巧这个话题很大， 对于练习时长不足两年半的我根本没法表述清楚，但是在日常的工作中，又会频繁得跟 C++ 打交道，这使我迫切得需要开这样一篇话题，记录并提炼出通过从书籍和日常工作中得到的知识和经验。好好利用好 C++ 这个强大的武器，使未来的工作或学习更加一帆风顺。

以下记录的，都是我在日常工作中「经常使用到的」，或者在「优化时发现能提升效率」的技巧。很可能技巧都比较简单，但是从我微不足道的经验来看，令人受益匪浅。

### P2 编译期 Assertions（断言）

随着泛型编程的大量使用，我们越来越需要更好的静态检查（static checking）以及更个性化的错误信息输出。

假使我们想要设计一个做 safe casting 的函数，也就是想要在讲一个类型转变成另一个类型时，所有的信息都能被保留。或者说，大类型不能被 cast 成小类型。

我们可以先简单设计出如下的模板函数：

```cpp
template <class To, class From>
To safe_reinterpret_cast(From from) {
    assert(sizeof(From) <= sizeof(To));
    return reinterpret_cast<To>(from);
}
```

假使我们使用同样的语法调用这个函数：

```cpp
int i = ...;
char* p = safe_reinterpret_cast<char*>(i);
```

显然，这段代码是想将 `int` 类型的数据 safe cast 成 `char*` 类型。不过这里有**几个**可以优化的地方：`assert()` 内部的结果其实是一个**编译期常量**，但是 `assert()` 函数只有在**运行期**才会报错，这可能会导致我们在将代码迁移到其他系统时，无法发现并定位错误。我们可以

1. 考虑如何将报错提前到编译期。

2. 考虑编译期报错时如何输出有效信息。 

Van Horn  曾经就考虑过这个问题，他依赖长度为 0 的数组是非法的原则，设计了一种编译期报错的方法：

```cpp
#define STATIC_CHECK(expr) {char unnamed[(expr) ? 1: 0]}
template <class To, class From>
To safe_reinterpret_cast(From from) {
    STATIC_CHECK(sizeof(From) <= sizeof(To));
    return interpret_cast<To>(From);
}
...
void* somePointer = ...;
char c = safe_reinterpret_cast<char>(somePointer);
```

如果系统指针比 char 要大，编译器就会报错，说我们尝试创造一个长度为 0 的数组。

不过这样问题也比较明显：错误信息太模糊了，而且很难提供可移植性的个性化错误信息。错误信息没有一定需要遵循的规则，这些规则都是由编译器决定。

更好的解决方法是去依赖一个有信息名字的模板，幸运的话，编译器会在错误信息中提到模板的名字。我们可以设计成如下情况

```cpp
template<bool> struct CompileTimeChecker {
    CompileTimeChecker(...);
};
template<> struct CompileTimeChecker<false> { };
#define STATIC_CHECK(expr, msg) \
{\
    class ERROR_##msg {}; \
    (void)sizeof(CompileTimeChecker<(expr) != 0>((ERROR_##msg()))); \
}


template <class To, class From>
To safe_reinterpret_cast(From from) {
    STATIC_CHECK(sizeof(From) <= sizeof(To),
        Destination_Type_Too_Narrow);
    return reinterpret_cast<To>(from);
}
...
void* somePointer = ...;
char c = safe_reinterpret_cast<char>(somePointer);
```

这样的话，报错信息就可能变成：`Error: Cannot convert ERROR_Destination_Type_Too_Narrow to CompileTimeChecker
<false>`.

### P0 模板偏特化

模板偏特化允许我们对模板可实例化集合中的一个子集进行特化。这也是在泛型编程中经常使用的技术。

下面通过一个例子去简单介绍什么是模板偏特化。

首先，我们假设有一个模板 `Widget`，它有两个类型参数，如下式：

```cpp
template <class Window, class Controller>
class Widget {
    // 泛型实现
    ...
}；
```

而后，我们可以对这个类进行显式的特化：

```cpp
template<>
class Widget<ModalDialog, MyController> {
    // 特化实现
    ...
};
```

在看到如上的特化定义后，无论我们在哪里定义一个类型为 `Widget<ModalDialog, MyController>` 的对象，编译器都会使用特化实现的版本，而对`Widget` 的其他实现，编译器则会使用泛型的实现。

然后，我们有时候想对任意的 `Window` 和 特定的 `Controller`(这里假设是 `MyController`) 进行特化实现，这时，我们就会使用到模板的偏特化：

```cpp
template<class Window>
class Widget<Window, MyController> {
    // 模板偏特化
};
```

特别地，在一个类模板的偏特化中，你只会指定一部分模板参数，而保持另一部分模板参数的泛化性。当我们实例化程序中的类模板时，编译器会尝试找到最匹配的。这个匹配算法是复杂且准确的，这意味着，即使我们已经有一个 `Widget` 在任意 `Window` 和指定的 `MyContorller` 的偏特化模板，我们还可以偏特化下面的类模板，且在编译时依然能保证准确性。

```cpp
template <class ButtonArg>
class Widget<Button<ButtonArg>, MyController> {
    // 进一步的模板偏特化
}
```

这确保了我们在实例化模板时的灵活性，但需要注意一点，我们不能偏特化函数：无论是成员函数还是非成员函数：

- 即使我们能完全特化一个类模板的成员函数，我们不能偏特化类成员函数（这里的为什么，等后续补充）

- 对命名空间级别的(非成员的)模板函数，我们也不能偏特化，但是我们可以做一个非常接近的操作，就是重载（overloading），在实际应用中，这意味着我们只能对函数的参数做细粒度的特化，而不是对返回值或内部使用类型，比如
  
  ```cpp
  template <class T, class U> T Fun(U obj); // 原始模板
  template <class U> void Fun<void, U>(U obj); // 非法偏特化, 不能对返回值做细粒度特化
  template <class T> T Fun(Window obj); // 合法重载
  ```

对编译器编写者而言，偏特化缺乏粒度的特性，显然让生活更容易，但是对开发者来说，却有不好的影响，后面的一个工具特别是为了减少这种偏特化的限制。

### P1 局部类

我们能在函数里定义类，比如：

```cpp
void Fun() {
    class Local {
        ... 成员变量 member variables ...
        ... 成员函数定义 member function definitions ...
    };
    ... 使用 Local 这个类的代码 ...
}
```

需要注意，局部类有一些限制：

1. 不能定义静态成员变量
2. 无法使用统一命名空间里的非静态局部变量

不过，这个局部类有趣的地方是：你能在模板函数中使用他们。特别的，在模板函数中定义的局部类能使用 enclosing 函数的模板变量。

下面这个模板函数 `MakeAdapter` 目的是让一个接口适用于另一个。有了局部类的帮助，`MakeAdapter` 及时实现了一个接口。局部类存储了泛型的成员。

```cpp
class Interface {
 public:
     virtual void Fun() = 0;
    ...
};

template <class T, class P>
Interface* MakeAdapter(const T& obj, const P& arg) {
    class Local : public Interface {
     public:
         Local(const T& obj, const P& arg)
            : obj_(obj), arg_(arg) {}
        virtual void Fun() {
            obj_.Call(arg_);
        }
     private:
        T obj_；
        P arg_;
    };
    return new Local(obj, arg);
}
```

局部类有一个独特的特征：他们是 final 的。外部用户不能从一个函数内部的类继承。如果没有局部类，我们不得不在一个隔离的翻译单元中增加一个未命名的命名空间。

### 将常实数映射成类型

## Reference

- [The C++ in-depth series] Alexandrescu, Andrei_Meyers, Scott_Vlissides, John - Modern C++ design_ generic programming and design patterns applied (2001_2011, Addison-Wesley Professional) 
- [C++之局部类](https://www.cnblogs.com/chmm/p/7469618.html)
