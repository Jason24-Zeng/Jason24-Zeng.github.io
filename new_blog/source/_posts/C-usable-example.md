---
title: C++ usable example
date: 2022-03-15 23:42:48
tags: 
- C++
- HandBook
categories:
- C++
password: 0418
top_img: /img/C-usable-example/C.png
cover: /img/C-usable-example/C.png
---

## 实现函数的 Join 功能

```c++
#include<algorithm>
#include<string>
#include<vector>
#include<iostream>
#include<iterator>
#include<sstream>

template<typename InputIterator>
std::string join(InputIterator first, InputIterator last,
                 const std::string& seperator = " ",
                 const std::string& concluder = "") {
    std::ostringstream ss;

    if (first != last) {
        ss << *first++;
    } 

    while (first != last) {
        ss << seperator;
        ss << *first++;
    }

    ss << concluder;
    return ss.str();
}


// 举个栗子，对某个二维数组进行输出
int main() {
    vector<vector<int>> matrix = {
        {1, 2, 3},
        {4},
        {},
        {5, 13, 16}
    };

    std::for_each(matrix.cbegin(), matrix.cend(),
                  [](std::vector<int>& a) {
        std::cout << join(a.cbegin(), a.cend()) << std::endl;
    });
}
```

我们同样可以使用 `ostringstream` 去替代 `stringstream`。这样的话，我们可以避免使用 `>>`。

```c++
#include<iterator>
#include<ostringstream>

template<typename InputIterator>
std::string join (InputIterator begin, InputIterator end,
                  const std::string& seperator = " ",
                  const std::string& concluder = "") {
    std::ostringstream ss;
    using value_type = typename std::iterator_traits<InputIterator>::value_type;
    std::copy(begin, end, std::ostream_iterator<value_type>(ss, seperator.c_str());
    
    ss << concluder;
    return ss.str();
}
```

### join `vector<string> ` to `string` with separater

```cpp
std::vector<std::string> strings;

const char* const delim = ", ";

std::ostringstream imploded;
std::copy(strings.begin(), strings.end(),
           std::ostream_iterator<std::string>(imploded, delim));
```

[C++ ostream out manipulation - Stack Overflow](https://stackoverflow.com/questions/5288396/c-ostream-out-manipulation/5289170#5289170)
