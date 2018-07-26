# 建立可迭代区域

我们已经认识了STL中提供的各种迭代器。我们只需实现一个迭代器，支持前缀加法`++`，解引用`*`和比较操作`==`，这样我们就能使用C++11基于范围的for循环对该迭代器进行遍历。

为了更好的了解迭代器，本节中将展示如何实现一个迭代器。迭代该迭代器时，只输出一组数字。实现的迭代器并不支持任何容器，以及类似的结构。这些数字是在迭代过程中临时生成的。

## How to do it...

本节中，我们将实现一个迭代器类，并且对该迭代器进行迭代：

1. 包含必要的头文件。

   ```c++
   #include <iostream> 
   ```

2. 迭代器结构命名为`num_iterator`:

   ```c++
   class num_iterator { 
   ```

3. 其数据类型只能是整型，仅用是用来计数的，构造函数会初始化它们。显式声明构造函数是一个好习惯，这就能避免隐式类型转换。需要注意的是，我们会使用`position`值来初始化`i`。这就让`num_iterator`可以进行默认构造。虽然我们的整个例子中都没有使用默认构造函数，但默认构造函数对于STL的算法却是很重要的。

   ```c++
   	int i;
   public:
   	explicit num_iterator(int position = 0) : i{position} {}
   ```

4. 当对迭代器解引用时*it`，将得到一个整数：

   ```c++
   	int operator*() const { return i; }
   ```

5. 前缀加法操作`++it`：

   ```c++
       num_iterator& operator++() {
           ++i;
           return *this;
       }
   ```

6. `for`循环中需要迭代器之间进行比较。如果不相等，则继续迭代：

   ```c++
       bool operator!=(const num_iterator &other) const {
       	return i != other.i;
       }
   };
   ```

7. 迭代器类就实现完成了。我们仍需要一个中间对象对应于` for (int i : intermediate(a, b)) {...}`写法，其会从头到尾的遍历，其为一种从a到b遍历的预编程。我们称其为`num_range`:

   ```c++
   class num_range {
   ```

8. 其包含两个整数成员，一个表示从开始，另一个表示结束。如果我们要从0到9遍历，那么a为0，b为10(`[0, 10)`)：

   ```c++
       int a;
       int b;
   public:
       num_range(int from, int to)
       	: a{from}, b{to}
       {}
   ```

9. 该类也只有两个成员函数需要实现：`begin`和`end`函数。两个函数都返回指向对应数字的指针：一个指向开始，一个指向末尾。

   ```c++
       num_iterator begin() const { return num_iterator{a}; }
       num_iterator end() const { return num_iterator{b}; }
   };
   ```

10. 所有类都已经完成，让我们来使用一下。让我们在主函数中写一个例子，遍历100到109间的数字，并打印这些数值：

   ```c++
   int main()
   {
       for (int i : num_range{100, 110}) {
       	std::cout << i << ", ";
       }
       std::cout << '\n';
   }
   ```

11. 编译运行后，得到如下输出：

    ```c++
    100, 101, 102, 103, 104, 105, 106, 107, 108, 109,
    ```

## How it works...

考虑一下如下的代码段：

```c++
for (auto x : range) { code_block; }
```

这段代码将被编译器翻译为类似如下的代码：

```c++
{
    auto __begin = std::begin(range);
    auto __end = std::end(range);
    for ( ; __begin != __end; ++__begin) {
        auto x = *__begin;
        code_block
    }
}
```

这样看起来就直观许多，也能清楚的了解我们的迭代器需要实现如下操作：

- operator!=
- operatpr++
- operator*

也需要`begin`和`end`方法返回相应的迭代器，用来确定开始和结束的范围。

> Note:
>
> 本书中，我们使用`std::begin(x)`替代`x.begin()`。如果有`begin`成员函数，那么`std::begin(x)`会自动调用`x.begin()`。当`x`是一个数组，没有`begin()`方法是，`std::begin(x)`会找到其他方式来处理。同样的方式也适用于`std::end(x)`。当用户自定义的类型不提供`begin/end`成员函数时，`std::begin/std::end`就无法工作了。

本例中的迭代器是一个前向迭代器。再来看一下使用`num_range`的循环，从另一个角度看是非常的简单。

> Note:
>
> 回头看下构造出迭代器的方法在`range`类中为`const`。这里不需要关注编译器是否会因为修饰符`const`而报错，因为迭代`const`的对象是很常见的事。

