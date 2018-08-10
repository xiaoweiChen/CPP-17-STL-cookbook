# 容器间相互复制元素

大多数STL数据结构都支持迭代器。这就意味着大多数数据结构能够通过成员函数`begin()`和`end()`成员函数得到相应的迭代器，并能对数据进行迭代。迭代的过程看起来是相同的，无论是什么样的数据结构都是一样的。

我们可以对`vector`，`list`，`deque`，`map`等等数据结构进行迭代。我们甚至可以使用迭代器作为文件/标准输入输出的出入口。此外，如之前章节介绍，我们能将迭代器接口放入算法中。这样的话，我们可以使用迭代器访问任何元素，并且可以将迭代器作为STL算法的参数传入，对特定范围内的数据进行处理。

`std::copy`算法可以很好的展示迭代器是如何将不同的数据结构进行抽象，而后将一个容器的数据拷贝到另一个容器。类似这样的算法就与数据结构的类型完全没有关系了。为了证明这点，我们会把玩一下`std::copy`。

## How to do it...

本节中，我们将对不同的变量使用`std::copy`。

1. 首先，包含必要的头文件，并声明所用到的命名空间。

   ```c++
   #include <iostream>
   #include <vector>
   #include <map>
   #include <string>
   #include <tuple>
   #include <iterator>
   #include <algorithm>
   
   using namespace std;
   ```

2. 我们将使用整型和字符串值进行组对。为了能很好的将其进行打印，我们将会重载`<<`流操作：

   ```c++
   namespace std {
   ostream& operator<<(ostream &os, const pair<int, string> &p)
   {
   	return os << "(" << p.first << ", " << p.second << ")";
   }
   }
   ```

3. 主函数中，我们将使用整型-字符串对填充一个`vector`。并且我们声明一个`map`变量，其用来关联整型值和字符串值：

   ```c++
   int main()
   {
       vector<pair<int, string>> v {
           {1, "one"}, {2, "two"}, {3, "three"},
           {4, "four"}, {5, "five"}};
       
       map<int, string> m;
   ```

4. 现在将`vector`中的前几个整型字符串对使用`std::copy_n`拷贝到`map`中。因为`vector`和`map`是两种完全不同的结构体，我们需要对`vector`中的数据进行变换，这里就要使用到`insert_iterator`适配器。`std::inserter`函数为我们提供了一个适配器。在算法中使用类似`std::copy_n`的算法时，需要与插入迭代器相结合，这是一种更加通用拷贝/插入元素的方式(从一种数据结构到另一种数据结构)，但这种方式不是最快的。使用指定数据结构的成员函数插入元素无疑是更加高效的方式：

   ```c++
   	copy_n(begin(v), 3, inserter(m, begin(m)));
   ```

5. 让我们打印一下`map`中的内容。纵观本书，我们会经常使用`std::copy`函数来打印容器的内容。`std::ostream_iterator`在这里很有用，因为其可以将用户的标准输出作为另一个容器，而后将要输出的内容拷贝过去：

   ```c++
       auto shell_it (ostream_iterator<pair<int, string>>{cout,
       ", "});
       
   	copy(begin(m), end(m), shell_it);
       cout << '\n';
   ```

6. 对`map`进行清理，然后进行下一步的实验。这次，我们会将`vector`的元素*移动*到`map`中，并且是所有元素：

   ```c++
       m.clear();
       
   	move(begin(v), end(v), inserter(m, begin(m)));
   ```

7. 我们将再次打印`map`中的内容。此外，`std::move`是一种改变数据源的算法，这次我们也会打印`vector`。这样，我们就会看到算法时如何对数据源进行的移动：

   ```c++
       copy(begin(m), end(m), shell_it);
       cout << '\n';
       
   	copy(begin(v), end(v), shell_it);
       cout << '\n';
   }
   ```

8. 编译运行这个程序，看看会发生什么。第一二行非常简单，其反应的就是`copy_n`和`move`算法执行过后的结果。第三行比较有趣，因为移动算法将其源搬移到`map`中，所以这时的`vector`是空的。在重新分配空间前，我们通常不应该访问成为移动源的项。但是为了这个实验，我们忽略它：

   ```c++
   $ ./copying_items
   (1, one), (2, two), (3, three),
   (1, one), (2, two), (3, three), (4, four), (5, five),
   (1, ), (2, ), (3, ), (4, ), (5, ),
   ```

## How it works...

`std::copy`是STL中最简单的算法之一，其实现也非常短。我们可以看一下等价实现：

```c++
template <typename InputIterator, typename OutputIterator>
OutputIterator copy(InputIterator it, InputIterator end_it,
OutputIterator out_it)
{
    for (; it != end_it; ++it, ++out_it) {
    	*out_it = *it;
    }
    return out_it;
}
```

这段代码很朴素，使用`for`循环将一个容器中的元素一个个的拷贝到另一个容器中。此时，有人就可能会发问："使用`for`循环的实现非常简单，并且还不用返回值。为什么要在标准库实现这样的算法？"，这是个不错的问题。

`std::copy`并非能让代码大幅度减少的一个实现，很多其他的算法实现其实非常复杂。这种实现其实在代码层面并不明显，但STL算法更多的在于做了很多底层优化，编译器会选择最优的方式执行算法，这些底层的东西目前还不需要去了解。

STL算法也让能避免让开发者在代码的可读性和优化性上做权衡。

> Note：
>
> 如果类型只有一个或多个(使用`class`或`struct`包装)的矢量类型或是类，那么其拷贝赋值通常是轻量的，所以可以使用`memcopy`或`memmove`进行赋值操作，而不要使用自定义的赋值操作符进行操作。

这里，我们也使用了`std::move`。其和`std::copy`一样优秀，不过`std::move(*it)`会将循环中的源迭代器，从局部值(左值)转换为引用值(右值)。这个函数就会告诉编译器，直接进行移动赋值操作来代替拷贝赋值操作。对于大多数复杂的对象，这会让程序的性能更好，但会破坏原始对象。

