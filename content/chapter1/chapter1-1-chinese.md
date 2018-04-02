# 使用结构化绑定来解包绑定的返回值

C++17配备了一种新的特性——**结构化绑定**，其可以结合语法糖来自动推到类型，并可以从组对、元组和结构体中提取单独的变量。其他编程语言中，这种特性也被成为**解包**。

## How to do it...

使用结构化绑定是为了能够更加简单的，为绑定了多个变量的结构体进行赋值。我们先来看下在C++17之前的标准如何完成这个功能。然后，我们将会看到一些使用C++17实现该功能的例子：

- 访问std::pair中的一个元素：假设我们有一个数学函数divide_remainder，需要输入一个除数和一个被除数作为参数，返回得到的分数的整数部分和余数。可以使用一个std::pair来绑定这两个值:

  std::pair<int, int> divide_remainder(int dividend, int divisor);

考虑使用如下的方式访问组对中单个的值：

```c++
const auto result (divide_remainder(16, 3));
std::cout << "16 / 3 is " <<
          << result.first << " with a remainder of "
          << result.second << '\n';
```

与上面的代码段不同，我们现在可以将相应的值赋予对应的变量，这样写出来的代码可读性更高:

```c++
auto [fraction, remainder] = divide_remainder(16, 3);
std::cout << "16 / 3 is "
          << fraction << " with a remainder of "
          << remainder << '\n';
```

- 也能对std::tuple进行结构化绑定：让我们使用下面的实例函数，获取股票的在线信息：

```c++
std::tuple<std::string, std::chrono::system_clock::time_point, unsigned>
stock_info(const std::string &name);
```

我们可以使用如下的方式获取这个例子的各个变量的值：

```c++
const auto [name, valid_time, price] = stock_info("INTC");
```



## How it works...



## There's more...



