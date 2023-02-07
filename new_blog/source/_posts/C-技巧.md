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

### P1 将实常数映射成类型

有一个非常简单的模板，对许多泛型编程 idioms（习语）有用：

```cpp
template <int v>
struct Int2Type {
    enum {value = v};
};
```

`Int2Type` 对输入的不同常实数会生成不同的类型，这是因为不同的模板实例化是不同的类型。另外，产生这个类型的值，会被保存在枚举成员 `value` 中

这个模板为什么被广泛应用呢？无论何时我们需要给一个常实数附以类型时，我们都可以使用它。通过这个方式，我们可以依赖编译期计算的结果，去选择不同的函数，从而可以有效得依赖一个常实数实现静态分派（static dispatch）

典型的，在如下两个情况都满足的时候，我们应该使用 `Int2Type` ：

1. 取决于一个编译期常数，我们会去挑选几个不同函数中的一个

2. 我们需要在编译期做这样的分派

对于运行期的分派，我们可以简单地使用 `if-else` 语句或者 `switch` 的语句。运行期的消耗在大多数情况下可忽略不计。但是，我们通常不这么做。（不这么做的原因不是运行期更耗时）`if-else` 语句要求两个分支都得成功编译，即使我们在编译期时就可以知道 `if` 条件的选择情况。

下面通过一个例子来解释上面这句话的意思。

假设我们正在设计一个泛型容器 `NiftyContainer`，这个容器被包含的类型模板化：

```cpp
template <class T> class NiftyContainer {
    ...
};
```

假使 `NiftyContainer` 容器包含了指向类型 `T` 的对象的指针。为了复制包含在容器中的对象，我们要么调用它自身的拷贝构造函数，要么调用一个为多态类型（polymorhic types）准备得虚函数 `Clone()`。至于选择哪种函数，我们可以通过一个布尔类型的模板参数获得信息。

为此，我们很容易想到以下的实现方法：

```cpp
template <typename T, bool isPolymorphic>
class NiftyContainer {
    ...
    void DoSomething() {
        T* pSomeObj = ...;
        if (isPolymorphic) {
            T* pNewObj = pSomeObj->Clone();
            // ...多态算法...
        } else {
            T* pNewObj = new T(*pSomeObj);
            // ...非多态算法...
        }
    }  
};
```

这样实现可能让编译器通不过编译。比如，多态算法要求使用 `Clone` 成员函数，如果任何类型没有定义一个这样的成员函数，`NiftyContainer::DoSomething` 将不能编译。的确我们在编译的时候已经能决定执行哪个 if 状态了，但是，编译器不会管这些，他会尝试编译两个分支，即使优化器后面会删除无用的代码。也就是说，我们可能会出现这样的情况：我们尝试调用 `NiftyContainer<int, false>` 的 `DoSomething` 函数，编译器却在 `pObj->Clone()` 处报错了。

另外，非多态分支的代码也可能编译失败，如果 `T` 是一个多态类型并且非多态代码分支尝试 `new` 一个 `T(*pObj)` ，这也可能导致代码编译失败：如果 `T` 禁用了拷贝构造函数或者让拷贝构造函数变成私有的。PS. 好的多态类就应该做这些考虑。

让编译器不用操心代码是否是无用，这个想法是好的，但是有更让人满意的解决方法，在使用这个简洁的解决方法的时候，我们会用到 `Int2Type`：它会根据布尔值 `isPolymorphic` 的不同，将其即时得转变成两个不同的类型，然后我们就可以利用简单的重载去使用 `Int2Type<isPolymorhic>` 啦。代码重构如下：

```cpp
template <typename T, bool isPolymorphic>
class NiftyContainer {
 private:
    void DoSomething(T* pObj, Int2Type<true>) {
        T* pNewObj = pObj->Clone();
        // ...多态算法...
    }
    void DoSomething(T* pObj, Int2Type<false>) {
        T* pNewObj = new T(*pSomeObj);
        // ...非多态算法...
    }
 public:
    void DoSomething(T* pObj) {
        DoSomething(pObj, Int2Type<isPolymorphic>());
    }
};
```

这个重载实现了两个需要的算法，这上面的 trick 点在于，因为编译器不会编译它们不适用的模板函数，它只检查它们语法。这样，我们就在编译期实现了模板代码的分派。

### 类型到类型的映射







## Reference

- [The C++ in-depth series] Alexandrescu, Andrei_Meyers, Scott_Vlissides, John - Modern C++ design_ generic programming and design patterns applied (2001_2011, Addison-Wesley Professional) 
- [C++之局部类](https://www.cnblogs.com/chmm/p/7469618.html)
