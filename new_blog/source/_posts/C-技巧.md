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

这样的话，报错信息就可能变成：

```cpp
Error: Cannot convert ERROR_Destination_Type_Too_Narrow to CompileTimeChecker<false>
```

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

### P1 类型到类型 Type2Type 的映射

前面已经提到，我们不能对模板函数进行偏特化，但是，有时我们又需要模拟相似的功能。参考下面函数，它会通过将参数传给构造函数的方式，创建一个新的对象。

```cpp
template <class T, class U>
T* Create(const U& arg) {
    return new T(arg);
}
```

现在假设我们的应用中有一个规则：

1. 类型为 `Widget` 的对象是不能碰的遗留代码，我们想要构建必须传入两个参数，第二个参数必须是一个固定值，比如 -1。

2. 从 `Widget` 继承过来的类，不会有这个问题。

现在的问题是：我们怎么特化 `Create` 来让它用和其他类型不同的方式处理 `Widget`? 显而易见的一种方法，是创造一个单独的`CreateWidget` 函数去解决特殊的问题。但是，你没有一个统一的接口去创建 `Widget`s 和它的继承对象，从而让 `Create` 在任何泛型代码中不好用。 

就像刚开始就提到的，我们不能偏特化一个函数，具体在这个例子中便是：

```cpp
template <class U>
Widget* Create(const U& arg) {
    return new T(arg, -1);
}
```

解决这一类问题仅有的工具是重载 overloading，一种办法是传一个类型为 T 的 dummy 对象：

```cpp
template <class T, class U>
T* Create(const U& arg, T /* dummy */) {
    return new T(arg);
}
template <class U>
Widget* Create(const U& arg, Widget /* dummy */) {
    return new Widget(arg, -1);
}
```

这样的解决方式很容易让我们创建一个随意的复杂对象，但是我们却永远不会使用它。我们需要个更轻便的载体去将关于 `T` 的类型信息传给 `Create`，这时候我们就可以考虑使用 `Type2Type`，一个类型的代表，一个可以传递给重载函数的轻标识符。

```cpp
template <typename T>
struct Type2Type {
    typedef T OriginalType;
}
```

可以看出 `Type2Type` 不包含任意值，但是有一个不同的类型引导实例化不同的 `Type2Type`，这正是我们想要的。有了这个工具，我们就可以轻松写下如下优化：

```cpp
template <class T, class U>
T* Create(const U& arg, Type2Type<T>) {
    return new T(arg);
}
template <class U>
Widget* Create(const U& arg, Type2Type<Widget>) {
    return new Widget(arg, -1);
}
// 使用 Create()
String* pStr = Create("Hello", Type2Type<String>());
Widget* pW = Create(100, Type2Type<Widget>());
```

`Create` 的第二个参数只是用来选择合理的重载，现在我们能对不同的 `Type2Type` 实例去特化 `Create`。



### P2 类型选择

有些泛型代码会根据一个布尔常数去选择两个类型中的一种。这一章就是要讨论这个的实现。

举个前面提到的例子 `NiftyContainer`，假设我们想要在底层存储中使用 `std::vector`。显然，对多态类型我们必须存储指针而非值本身，而对非多态类型，我们可能想要存储值，因为这更加方便高效。

首先是 `NifityContainer` 这个类：

```cpp
template <typename T, bool isPolymorphic>
class NiftyContainer {
    ...
};
```

我们需要存储要么一个 `vector<T*>` （`isPolymorphic` 为 `true`）要么一个 `vector<T>` （`isPolymorphic` 为 `false`）。总之，我们需要根据 `isPolymorphic` 的值决定定义的  `ValueType`要么是 `T*`，要么是`T`。

我们可以定义一个如下的特征类模板解决问题：

```cpp
template <typename T, bool isPolymorphic>
struct NiftyContainerValueTraits {
    typedef T* ValueType;
};
template <typename T>
struct NiftyContainerValueTraits<T, false> {
    typedef T ValueType;
};
template <typename T, bool isPolymorphic>
class NiftyContainer {
    ...
    typedef NiftyContainerValueTraits<T, isPolymorphic>
        Traits;
    typedef typanme Traits::ValueType ValueType;
};
```

这种处理问题的方式显得不必要的笨重，并且，他没法扩展，每次类型选择，我们都必须定义一个信息的特征类型模板。

库类模板 `Select` 给我们提供了一个能完美解决问题的类型选择方法。它的定义使用了模板偏特化：

```cpp
template <bool flag, typename T, typename U>
struct Select {
    typedef T Result;
};
template <typename T, typename U>
struct Select<false, T, U> {
    typedef U Result;
};
```

这里解释一下这个 `Select` 怎么工作的。

1. 当 `flag` 为 `true` 时，编译器会使用第一个泛型定义因此 `Result` 等于 `T`

2. 当 `flag` 为 `false` 时，特化的模板被使用，因此 `Result` 等于 `U`

现在，我们就能更简单得定义 `NiftyContainer::ValueType` 了，而且也更容易扩展了

```cpp
template <typename T, bool isPolymorphic>
class NiftyContainer {
    ...
    typedef typename Select<isPolymorphic, T*, T>::Result
        ValueType;
    ...
};
```

### P1 在编译期检测可转化性和继承性

我们在实现模板函数和模板类时，经常会想一个问题：给定随意两种类型 `T` 和 `U` ，我们该怎么检测是否 `U` 为 `T` 的继承呢？在编译期发现这样的关系是在泛型库类实现高级优化的关键。在一个泛型函数中，如果我们能确定这个类实现了某一个特殊的接口，我们就可以根据这个信息去采用最佳算法。在编译器发现这个意味着我们不用使用 在编译期及其耗时的`dynamic_cast`。

发现继承关系依赖一个甄别可转化性的更一般化的机制。而更一般的问题是，我们要怎么检测随机的一个类型 `T` 是否支持向 `U` 的自动转化。

有一种解决方法，它会依赖 `sizeof` 的实现。`sizeof` 有着许多令人吃惊的力量：我们可以对任何表达式使用`sizeof`，不管这个表达式多么的复杂，它能返回表达式的大小，且不用拖到运行期。这意味着 `sizeof` 能意识到重载，模板实例化，转化规则等一切参与 C++ 表达式的东西。事实上，`sizeof` 背后暗含一个用来推到表达式类型的完整设施。最终，`sizeof` 会丢弃表达式并且只返回结果的大小。

检测转换能力 的想法依赖于 `sizeof` 与 重载函数的共用。我们提供一个函数的两个重载，一个接受类型，并且将其转化成 `U`，另一个接受其他任何类型。我们用一个临时的类型 `T` 去调用重载函数，而`T` 对 `U` 的可转换能力正式我们想要判断的。如果前一个调用 `U` 的函数被调用，那我们知道 `T` 能转化成`U`，如果的另一个函数被调用，那么 `T` 就不能被转化成 `U`。我们设计两个重载函数返回不同大小的类型，然后便可以使用`sizeof` 去判断了。这两个类型本身不重要，只要它们大小不同就行。

ok，解释到这里了，我们来具体实现以下它。

首先，我们创建两个不同大小的类型，显然 `char` 和 `long double` 有不同的大小，但是 C++ 标准并不保证一定不同，一个傻瓜式的定义如下

```cpp
typedef char Small;
class Big {char dummy[2]};
```

根据定义，`sizeof(Small)` 为 1，`Big` 的大小未知，但一定大于 1，这是我们唯一能保证的东西。

下一步，我们需要两个重载函数，一个接受 `U` 并且返回一个 `Small`

```cpp
Small Test(U);
```

我们怎么写一个函数接受其他任何呢？一个模板不是解决方法，因为模板总是要求最佳匹配条件，因此掩盖可转化的类型。我们需要一个匹配，这个匹配会比自动转化更糟糕，也就是，转化只会在没有自动转化时发生。我很快看了一下施行于函数调用的转化规则，然后发现了所谓的省略符匹配规则（ellipsis match），这时最差的匹配了，也是我们正好希望的：

```cpp
Big Test(...);
```

传一个 C++ 的对象到有省略符的函数会有未定义的结果，但这不重要。事实上没有东西会真的调用这个函数，它甚至不会实现。想想 `sizeof` 实际不会衡量函数的入参。

现在，我们需要实施 `sizeof` 到 `Test` 的调用，给它传递一个 `T` 作为入参。

```cpp
const bool convExists = sizeof(Test(T())) == sizeof(Small);
```

对，`Test` 的调用得到一个默认构造对象 `T()`，然后 `sizeof` 会分离出表达式结果的大小，它要么是 `sizeof(Small)` 要么是 `sizeof(Big)`，这取决于是否编译器能发现一种转化。

有一个小问题，如果 `T` 的默认构造函数是私有的，表达式 `T()` 会在编译时失败，我们的所有努力都会没有回报，不够新云的是，有一个非常简单的解决方法，就是使用一个稻草人函数（strawman function），它返回一个 T 对象（记住，我们在处于 `sizeof` 的神奇世界中，没有表达式会被实际求值）。因此，编译器和我们都会非常满意。

```cpp
T MakeT();  // 没有实现，不止不会做任何是，甚至在运行期都没有真实存在过
const bool convExists = sizeof(Test(MakeT())) == sizeof(Small);
```

现在我们把这个函数类的所有东西包裹在一起，隐藏类型推导的所有细节，只展现结果：

```cpp
template <class T, class U>
class Coversion {
    typedef char Small;
    class Big {char dummy[2];};
    static Small Test(U);
    static Big Test(...);
    static T MakeT();
 public:
    enum {
        exists = sizeof(Test(MakeT())) == sizeof(Small);
    };
};
```

现在，我们可以测试 `Coversion` 类模板：

```cpp
int main() {
    using namespace std;
    cout << Conversion<double, int>::exists << ' '
         << Conversion<char, char*>::exists << ' '
         << Conversion<size_t, vector<int>>::exists << ' ';
}
```

这个小程序会打印 `1 0 0`，注意虽然 `vector` 的确实现了一个构造函数，这个函数获取一个入参 `size_t`，但是这个转化测试返回 0，因为那个构造函数时 explicit 的。explicit 构造函数时没法担任转换函数的。

我们能在 `Coversion` 中增加一个常数 `sameType`，如果 `T` 和 `U` 表示同一个类型时为 `true`：

```cpp
template <class T, class U>
class Conversion {
    ...同上...
    enum {sameType = false};
}
```

我们通过 `Conversion` 的一个片特护来实现 `sameType`：

```cpp
template <class T>
class Conversion<T, T>
{
 public:
    enum {exists = 1, sameType = 1};
};
```

最后，有了 `Conversion` 的帮助，我们现在可以非常容易得定义继承了：

```cpp
#define SUPERSUBCLASS(T, U) \
    (Conversion<const U*, const T*>::exists && \
    !Conversion<const T*, const void*>::sameType)
```

如果 `U` 是 public 继承自 `T`,  或者 `T` 和 `U` 是同一类型的，`SUPERSUBCLASS(T, U)`会传入 `true`。当 `SUPERSUBCLASS` 对 `const T*` 和 `const U*` 做可转化评估时，只有三种情况 `const U*` 能隐式转化成 `const T*`：

1. `T`  和 `U` 同类型

2. `T` 是 `U` 的模糊 public 基类

3. `T` 是 `void`

最后一种情况被第二个测试消除了。实际上，接受第一种情况作为 `is-a` 的退化情况是非常有用的，因为在实际使用中，我们经常会考虑一个类是它自己的 superclass。如果我们需要更严格的测试，可以这么写：

```cpp
#define SUPERSUBCLASS_STRICT(T, U) \
    (SUPERSUBCLASS(T, U) && \
    !Conversion<const T, const U>::sameType)
```

为什么代码都加上`const` 修饰？因为我们不想因为 `const` 导致转型失败，如果模板代码实施了 两次 `const`，第二个 `const` 会被忽略掉，总之，保险起见，我们在 `SUPERSUBCLASS` 中都是用 `const`。



### P2 一个关于 `type_info` 的外覆类（wrapper）

标准 C++ 提供了一个类：`std::type_info` ，它能让我们在运行期调查类的类型。通常 `type_info` 会和 `typeid` 操作符并用，后者会传一个指向 `type_info` 对象的引用：

```cpp
void Fun(Base* pObj) {
    // 比较两个 type_info 对象，它们分别与 *pObj 和 Derived 有关
    if (typeid(*pObj) == typeid(Derived)) {
        // pObj 事实上指向一个 Derived 对象
    }
    ...
}
```

`typeid` 操作符返回一个指向类型为 `type_info` 对象的引用。除了支持比较算法 `operator==` 和 `operator!=` 以外，`type_info`提供了额外的两个函数。

- `name` 成员函数，以 `const char*` 的格式，返回一个类型的文本表达。因为没有将类名映射成字符串的标准方法，所以我们不应该期望 `typeid(Widget)` 返回 "Widget" 这样的字符串。一个比较令人满意的做法是让 `type_info:name` 对所有类型返回空字符串。

- `before` 成员函数，它给 `type_info` 对象带来了一种次序关系。使用 `type_info::before` ，我们能对 `type_info` 对象建立索引

不幸的是，`type_info` 的好用功能被包装得难以使用。`type_info` 类将拷贝构造函数和拷贝赋值操作符禁用，这使得存储 `type_info` 对象变得不可能。不过我们可以存储指向 `type_info` 对象的指针。这个通过 `typeid` 返回的对象，采用了 static 的存储方式，因此我们不用担心其寿命，但我们得关注指针间的辨识。

C++ 标准不能保证每次调用 `typeid(int)` 能返回指向同一个 `type_info` 对象的引用。如此一来，我们没法比较指向 `type_info` 对象的指针了。为此，我们必须将指针存储到 `type_info` 对象中，并且通过运用 `type_info::operator==` 到解引用的方式，进行指针的比较。

如果我们想要对 `type_info` 对象进行排序，我们事实上必须存储指向 `type_info` 的指针，并且这次我们必须使用 `before` 这个成员函数。这样一来，如果我们想要使用 STL 的有序容器，必须写一个小的仿函数 `functor` 并处理指针。

这些笨拙到足以让我们委派一个关于 `type_info` 的外覆类来存储指向一个 `type_info` 的指针，并且，这个外覆类还需要提供：

- 所有 `type_info` 的成员函数

- value 的语义（与 reference 相对），即 public 的拷贝构造函数和拷贝赋值操作符

- 定义 `operator<` 和 `operator==` 来保证比较的无暇

Loki 定义了一个外覆类 `TypeInfo`，它实现了一个与 `type_info` 有关的，非常方便的外覆类，其大纲如下：

```cpp
class TypeInfo {
  public:
    TypeInfo();
    TypeInfo(const std::type_info&);
    TypeInfo(const TypeInfo&);
    TypeInfo& operator=(const TypeInfo&);
    bool before(const TypeInfo&) const;
    const char* name() const;
 private:
    const std::type_info* pInfo_;  
};
// 比较操作符
bool operator==(const TypeInfo&, const TypeInfo&);
bool operator!=(const TypeInfo&, const TypeInfo&);
bool operator<(const TypeInfo&, const TypeInfo&);
bool operator>(const TypeInfo&, const TypeInfo&);
bool operator<=(const TypeInfo&, const TypeInfo&);
bool operator>=(const TypeInfo&, const TypeInfo&);
```

由于其转换构造函数接受一个 `std::type_info` 的入参，我们能直接比较类型为 `TypeInfo` 和 `std::type_info` 的对象，如下：

```cpp
void Fun(Base* pObj) {
    TypeInfo info = typeid(Derived);
    ...
    if (typeid(*pObj) == info) {
        // pBase 事实上指向一个 Derived 对象
    }
    ...
}
```

拷贝和比较 `TypeInfo` 对象在许多情况下都非常重要。

### `NullType` 和 `EmptyType`



## Reference

- [The C++ in-depth series] Alexandrescu, Andrei_Meyers, Scott_Vlissides, John - Modern C++ design_ generic programming and design patterns applied (2001_2011, Addison-Wesley Professional) 
- [C++之局部类](https://www.cnblogs.com/chmm/p/7469618.html)
