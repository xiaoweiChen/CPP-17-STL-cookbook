# 使用结构化绑定来解包绑定的返回值

C++17配备了一种新的特性——**结构化绑定**，其可以结合语法糖来自动推到类型，并可以从组对、元组和结构体中提取单独的变量。其他编程语言中，这种特性也被成为**解包**。

## How to do it...

使用结构化绑定是为了能够更加简单的，为绑定了多个变量的结构体进行赋值。我们先来看下在C++17标准之前是如何完成这个功能的。然后，我们将会看到一些使用C++17实现该功能的例子：

- 访问`std::pair`中的一个元素：假设我们有一个数学函数`divide_remainder`，需要输入一个除数和一个被除数作为参数，返回得到的分数的整数部分和余数。可以使用一个`std::pair`来绑定这两个值:

  `std::pair<int, int> divide_remainder(int dividend, int divisor);`

考虑使用如下的方式访问组对中的单个值：

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

- 也能对`std::tuple`进行结构化绑定：让我们使用下面的实例函数，获取股票的在线信息：

```c++
std::tuple<std::string, std::chrono::system_clock::time_point, unsigned>
stock_info(const std::string &name);
```

我们可以使用如下的方式获取这个例子的各个变量的值：

```c++
const auto [name, valid_time, price] = stock_info("INTC");
```

* 结构化绑定也能用在自定义结构体上。假设有这么一个结构体：

```c++
struct employee{
    unsigned id;
    std::string name;
    std::string role;
    unsigned salary;
};
```

现在我们来看下如何使用结构化绑定访问每一个成员。我们假设有一组`employee`结构体的实例，存在于`vector`中，下面使用循环将其内容进行打印：

```c++
int main(){
    std::vector<employee> employees{
        /* Initialized from somewhere */
    };
    
    for (const auto &[id, name, role, salary] : employees){
        std::cout << "Name: " << name
                  << "Role: " << role
                  << "Salary: " << salary << '\n';
    }
}
```

## How it works...

结构化绑定以以下方式进行应用：

`auto [var1, var2, ...] = <pair, tuple, struct, or array expression>;`

- `var1, var2, ...`表示一个变量列表，其变量数量必须匹配表达式所对应的结构。
- `<pair, tuple, struct, or array expression>`必须是下面的其中一种：
  - 一个`std::pair`实例。
  - 一个`std::tuple`实例。
  - 一个结构体实例。其所有成员都必须是非静态成员，每个成员以基础类定义。结构体中的第一个声明成员赋予第一个变量的值，第二个声明的编程赋予第二个变量的值，依次类推。
  - 固定长度的数组。
- `auto`部分，也就是`var`的类型，可以是`auto`,`const auto`,`const auto&`和`auto&&`。

> Note:
>
> 不仅为了性能，还必须确保在适当的时刻使用引用，尽量减少不必要的副本。

如果中括号中变量不够，那么编译器将会报错:

```c++
std::tuple<int, float, long> tup(1, 2.0, 3);
auto [a, b] = tup; // Does not work
```

这个例子中想要将三个成员值，只赋予两个变量。编译器会立即发现这个错误，并且提示我们:

```
error: type 'std::tuple<int, float, long>' decomposes into 3 elements, but only 2 names were provided
auto [a, b] = tup;
```

## There's more...

STL中的基础数据结构都能通过结构结构化绑定直接进行访问，而无需修改任何东西。考虑下面这个例子，循环中打印`std::map`中的元素：

```c++
std::map<std::string, size_t> animal_population {
  {"humans", 7000000000},
  {"chickens", 17863376000},
  {"camels", 24246291},
  {"sheep", 1086881528},
  /* ... */
};

for (const auto &[species, count] : animal_population) {
  std::cout << "There are " << count << " " << species
            << " on this planet.\n";
}
```

从`std::map`容器中获取元素的方式比较特殊，我们会在每次迭代时获得一个`std::pair<const key_type, value_type>`实例。另外每个实例都需要进行结构化绑定(`key_type`绑定到`species`字符串上，`value_type`为一个`size_t`格式的统计数字)，从而达到访问每一个成员的目的。

在C++17之前，使用`std::tie`可达到类似的效果:

```c++
int remainder;
std::tie(std::ignore, remainder) = divide_remainder(16, 5);
std::cout << "16 % 5 is " << remainder << '\n';
```

这个例子展示了如何将结果组对解压到两个变量中。`std::tie`的能力远没有结构化绑定强，因为在进行赋值的时候，所有变量需要提前定义。另外，本例也展示了一种在`std::tie`中有，而结构化绑定没有的功能：可以使用`std::ignore`的值，作为虚拟变量。分数部分将会赋予到这个虚拟变量中，因为这里我们不需要用到分数值，所以使用虚拟变量忽略分数值。

> Note:
>
> 使用结构化绑定时，就不能再使用std::tie创建虚拟变量了，所以我们不得不绑定所有值到命名过的变量上。对部分成员进行绑定的做法是高效的，因为编译器可以很容易的对未绑定的变量进行优化。

回到之前的例子，`divide_remainder`函数也可以通过使用传入输出参数的方式进行实现：

```c++
bool divide_remainder(int dividend, int divisor, int &fraction, int &remainder);
```

调用该函数的方式如下所示：

```c++
int fraction, remainder;
const bool success {divide_remainder(16, 3, fraction, remainder)};
if (success) {
  std::cout << "16 / 3 is " << fraction << " with a remainder of "
            << remainder << '\n';
}
```

很多人都很喜欢使用特别复杂的结构，比如组对、元组和结构体，他们认为这样避免了中间拷贝过程，所以代码会更快。对于现代编译器来说，这种想法不再是正确的了，这里编译器并没有刻意避免拷贝过程，而是优化了这个过程。(其实拷贝过程还是存在的)。

> Note:
>
> 与C的语法特征不同，将复杂结构体作为返回值传回会耗费大量的时间，因为对象需要在返回函数中进行初始化，之后将这个对象拷贝到相应容器中返回给调用端。现代编译器支持**[返回值优化](https://zh.wikipedia.org/wiki/%E8%BF%94%E5%9B%9E%E5%80%BC%E4%BC%98%E5%8C%96)**(RVO, *return value optimization*)技术，这项技术可以省略中间副本的拷贝。