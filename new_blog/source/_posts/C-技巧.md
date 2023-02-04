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



### 编译期 Assertions（断言）

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



## Reference

- [The C++ in-depth series] Alexandrescu, Andrei_Meyers, Scott_Vlissides, John - Modern C++ design_ generic programming and design patterns applied (2001_2011, Addison-Wesley Professional) 
