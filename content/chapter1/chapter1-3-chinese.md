# 新的括号初始化规则

C++11引入了新的括号初始化语法`{}`。其不仅允许集合式初始化，而且还是对常规构造函数的调用。遗憾的是，当与`auto`类型变量结合时，这种方式就很容易出现错误。C++17将会增强这一系列初始化规则。本节中，我们将了解到如何使用C++17语法正确的初始化变量。

## How to do it...

一步初始化所有变量。使用初始化语法时，注意两种不同的情况：

- 不使用auto声明的括号初始化：

```c++
// Three identical ways to initialize an int:
int x1 = 1;
int x2{1};
int x3(1);

std::vector<int> v1{1, 2, 3}; // Vector with three ints
std::vector<int> v2 = {1, 2, 3}; // same here
std::vector<int> v3(10, 20); // Vector with 10 ints, each have value 20
```

- 使用auto声明的括号初始化：

```c++
auto v {1}; // v is int
auto w {1, 2}; // error: only single elements in direct
              // auto initialization allowed! (this is new)
auto x = {1}; // x is std::initializer_list<int>
auto y = {1, 2}; // y is std::initializer_list<int>
auto z = {1, 2, 3.0}; // error: Cannot deduce element type
```

## How it works...

无`auto`类型声明时，`{}`的操作没什么可大惊小怪的。当在初始化STL容器时，例如`std::vector`，`std::list`等等，括号初始化就会去匹配`std::initializer_list`(初始化列表)的构造函数，从而初始化容器。其构造函数会使用一种“贪婪”的方式，这种方式就意味着不可能匹配非聚合构造函数(与接受初始化列表的构造函数相比，非聚合构造函数是常用构造函数)。

`std::vector`就提供了一个特定的非聚合构造函数，其会使用任意个相同的数值填充`vector`容器：`std::vector<int> v(N, value)`。当写成`std::vector<int> v{N, value}`时，就选择使用`initializer_list`的构造函数进行初始化，其会将`vector`初始化成只有N和value两个元素的变量。这个“陷阱”大家应该都知道。

`{}`与`()`调用构造函数初始化的方式，不同点在于`{}`没有类型的隐式转换，比如`int x(1.2);`和`int x = 1.2;`通过静默的对浮点值进行向下取整，然后将其转换为整型，从而将x的值初始化为1。相反的，`int x{1.2};`将会遇到编译错误，初始化列表中的初始值，需要与变量声明的类型完全匹配。

> Note:
>
> 哪种方式是最好的初始化方式，目前业界是有争议的。括号初始化的粉丝们提出，使用括号的方式非常直观，直接可以调用构造函数对变量进行初始化，并且代码行不会做多于的事情。另外，使用{}括号将会是匹配构造函数的唯一选择，这是因为使用()进行初始化时，会尝试匹配最符合条件的构造函数，并且还会对初始值进行类型转换，然后进行匹配(这就会有处理构造函数二义性的麻烦)。

C++17添加的条件也适用于auto(推断类型)——C++11引入，用于正确的推导匹配变量的类型。`auto x{123};`中`std::initializer_list<int>`中只有 一个元素，这并不是我们想要的结果。C++17将会生成一个对应的整型值。

经验法则：

- `auto var_name {one_element};`将会推导出var_name的类型——与one_element一样。
- `auto var_name {element1, element2, ...};`是非法的，并且无法通过编译。
- `auto var_name = {element1, element2, ...};`将会使用`std::initializer_list<T>`进行初始化，列表中elementN变量的类型均为T。

C++17加强了初始化列表的鲁棒性。

> Note:
>
> 使用C++11/C++14模式的编译器解决这个问题时，有些编译器会将`auto x{123};`的类型推导成整型，而另外一些则会推导成 `std::initializer_list<int>`。所以，这里需要特别注意，编写这样的代码，可能会导致有关可移植性的问题！







