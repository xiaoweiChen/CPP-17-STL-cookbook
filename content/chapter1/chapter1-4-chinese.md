# 构造函数自动推导模板的类型

C++中很多类都需要指定类型，其实这个类型可以从用户所调用的构造函数中推导出来。不过，在C++17之前，这是一个未标准化的特性。C++17能让编译器自动的从所调用的构造函数，推导出模板类型。

## How to do it...

使用最简单的方法创建`std::pair`和`std::tuple`实例。其可以实现一步创建。

```c++
std::pair my_pair (123, "abc"); // std::pair<int, const char*>
std::tuple my_tuple (123, 12.3, "abc"); // std::tuple<int, double, const char*>
```

## How it works...

让我们定义一个类，了解自动化的对模板类型进行推断的价值。

```c++
template <typename T1, typename T2, typename T3>
class my_wrapper {
  T1 t1;
  T2 t2;
  T3 t3;
public:
  explicit my_wrapper(T1 t1_, T2 t2_, T3 t3_)
  : t1{t1_}, t2{t2_}, t3{t3_}
  {}
/* ... */
};
```

好！我们定义了一个模板类。C++17之前，我们为了创建该类的实例：

```c++
my_wrapper<int, double, const char *> wrapper {123, 1.23, "abc"};
```

我们省略模板特化的部分：

```c++
my_wrapper wrapper {123, 1.23, "abc"};
```

C++17之前，我们可能会通过以下的方式实现一个工厂函数：

```c++
my_wrapper<T1, T2, T3> make_wrapper(T1 t1, T2 t2, T3 t3)
{
  return {t1, t2, t3};
}
```

使用工厂函数：

```c++
auto wrapper (make_wrapper(123, 1.23, "abc"));
```

> Note:
>
> STL中有很多工厂函数，比如`std::make_shared`、`std::make_unique`、`std::make_tuple`等等。C++17中，这些工厂函数就过时了。当然，考虑到兼容性，这些工厂函数在之后还会保留。



## There's more...

我们已经了解过*隐式模板类型推导*。但一些例子中，不能依赖类型推导。如下面的例子：

```c++
// example class
template <typename T>
struct sum{
    T value;
    
    template <typename ... Ts>
    sum(Ts&& ... values) : value{(values + ...)} {}
};
```

结构体中，`sum`能接受任意数量的参数，并使用折叠表达式将它们添加到一起(本章稍后的一节中，我们将讨论折叠表达式，以便了解折叠表达式的更多细节)。加法操作后得到的结果保存在`value`变量中。现在的问题是，`T`的类型是什么？如果我们不显式的进行指定，那就需要通过传递给构造函数的变量类型进行推导。当我们提供了多个字符串实例，其类型为`std::string`。当我们提供多个整型时，其类型就为`int`。当我们提供多个整型、浮点和双浮点时，编译器会确定哪种类型适合所有的值，而不丢失信息。为了实现以上的推导，我们提供了*指导性显式推导*：

```c++
template <typename ... Ts>
sum(Ts&& ... ts) -> sum<std::common_type_t<Ts...>>;
```

指导性推导会告诉编译器使用`std::common_type_t`的特性，其能找到适合所有值的共同类型。来看下如何使用：

```c++
sum s {1u, 2.0, 3, 4.0f};
sum string_sum {std::string{"abc"}, "def"};
std::cout << s.value << '\n'
          << string_sum.value << '\n';
```

第1行中，我们创建了一个`sum`对象，构造函数的参数类型为`unsigned`, `double`, `int`和`floa`t。`std::common_type_t`将返回`double`作为共同类型，所以我们获得的是一个`sun<double>`实例。第2行中，我们创建了一个`std::string`实例和一个C风格的字符串。在我们的指导下，编译器推导出这个实例的类型为`sum<std::string>`。

当我们运行这段代码时，屏幕上会打印出10和abcdef。其中10为数值`sum`的值，abcdef为字符串`sum`的值。


