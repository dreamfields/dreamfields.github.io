---
title: Games101中的C++知识
tags:
  - C++
categories:
  - C++
date: 2022-10-23 10:25:09
---

# std::optional（C++17）

#### 框架代码：

```cpp
// games101/homework5/Renderer.cpp
std::optional<hit_payload> trace(
    const Vector3f &orig,
    const Vector3f &dir,
    const std::vector<std::unique_ptr<Object> > &objects){...}
```

#### 用途：

**使 C++ 的函数返回多个值**

#### 示例：

```cpp
#include <iostream>
#include <optional>

using namespace std;

struct Out {
    string out1 { "" };
    string out2 { "" };
};

optional<Out> func(const string& in) {
    Out o;
    if (in.size() == 0)
    // std::nullopt 是 C++ 17 中提供的没有值的 optional 的表达形式，等同于 { } 。
        return nullopt;
    o.out1 = "hello";
    o.out2 = "world";
    return { o };
}

// 等价于如下函数
// 这种做法可以做到让返回值更富有语意，并且可以很方便的扩展，如果要增加一个新的返回值，只需要扩展现有的结构体就可以了。
pair<bool, Out> func(const string& in) {
    Out o;
    if (in.size() == 0)
        return { false, o };
    o.out1 = "hello";
    o.out2 = "world";
    return { true, o };
}

int main() {
    if (auto ret = func("hi"); ret.has_value()) {
        cout << ret->out1 << endl;
        cout << ret->out2 << endl;
    }
    return 0;
}
```

#### 创建 optional

```cpp
// 空 optiolal
optional<int> oEmpty;
optional<float> oFloat = nullopt;

optional<int> oInt(10);
optional oIntDeduced(10);  // type deduction

// make_optional
auto oDouble = std::make_optional(3.0);
auto oComplex = make_optional<complex<double>>(3.0, 4.0);

// in_place
optional<complex<double>> o7{in_place, 3.0, 4.0};

// initializer list
optional<vector<int>> oVec(in_place, {1, 2, 3});

// 拷贝赋值
auto oIntCopy = oInt;
```

#### 访问 optional 对象

```cpp
// 跟迭代器的使用类似，访问没有 value 的 optional 的行为是未定义的
cout << (*ret).out1 << endl;
cout << ret->out1 << endl;

// 当没有 value 时调用该方法将 throws std::bad_optional_access 异常
cout << ret.value().out1 << endl;

// 当没有 value 调用该方法时将使用传入的默认值
Out defaultVal;
cout << ret.value_or(defaultVal).out1 << endl;
```
