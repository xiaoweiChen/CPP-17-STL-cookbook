# 容器间相互复制元素

大多数STL数据结构都是支持迭代器的。这就意味着大多数数据结构能够通过成员函数begin()和end()成员函数得到相应的迭代器，并能对数据进行迭代。迭代的过程看起来是相同的，无论是什么样的数据结构都是一样的。

我们可以对vector，list，deque，map等等数据结构进行迭代。我们甚至可以使用迭代器作为文件/标准输入输出的出入口。此外，如我们在之前章节介绍的，我们设置能将迭代器接口放入算法中。这样的话，我们可以使用迭代器访问任何元素，并且可以将迭代器作为STL算法的参数传入，对特定范围内的数据进行处理。

std::copy算法可以很好的展示迭代器是如何将不同的数据结构进行抽象，而后将一个容器的数据拷贝到另一个容器。类似这样的算法就与数据结构的类型完全没有关系了。为了证明这点，我们会把玩一下std::copy。

## How to do it...

本节，我们将对不同的变量使用std::copy。

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

3. 在主函数中，我们将使用整型-字符串对填充一个vector。并且我们声明一个map变量，其用来关联整型值和字符串值：

   ```c++
   int main()
   {
       vector<pair<int, string>> v {
           {1, "one"}, {2, "two"}, {3, "three"},
           {4, "four"}, {5, "five"}};
       
       map<int, string> m;
   ```

4. 现在，我们使用std::copy_n将vector中的前几个整型字符串对拷贝到map中。因为vector和map是两种完全不同的结构体，我们需要对vector中的数据进行变换，这里就要使用到insert_iterator适配器。std::inserter函数为我们提供了一个适配器。在算法中使用类似std::copy_n的算法时，需要与插入迭代器相结合，这是一种更加通用拷贝/插入元素的方式(从一种数据结构到另一种数据结构)，但这种方式不是最快的。使用指定数据结构的成员函数插入元素无疑是更加高效的方式：

   ```c++
   	copy_n(begin(v), 3, inserter(m, begin(m)));
   ```

5. 让我们打印一下map中的内容。纵观本书，我们会经常使用std::copy函数来打印容器的内容。std::ostream_iterator在这里很有用，因为其可以将用户的标准输出作为另一个容器，而后将要输出的内容拷贝过去：

   ```c++
       auto shell_it (ostream_iterator<pair<int, string>>{cout,
       ", "});
       
   	copy(begin(m), end(m), shell_it);
       cout << '\n';
   ```

6. 对map进行清理，然后进行下一步的实验。这次，我们会将vector的元素*移动*到map中，并且是所有元素：

   ```c++
       m.clear();
       
   	move(begin(v), end(v), inserter(m, begin(m)));
   ```

7. 我们将再次打印map中的内容。此外，std::move是一种改变数据源的算法，当然这次我们也会打印vector。这样，我们就会看到算法时如何对数据源进行的移动：

   ```c++
       copy(begin(m), end(m), shell_it);
       cout << '\n';
       
   	copy(begin(v), end(v), shell_it);
       cout << '\n';
   }
   ```

8. 让我们编译运行这个程序，看看会发生什么。第一二行非常简单，其反应的就是copy_n和move算法执行过后的结果。第三行比较有趣，因为移动算法将其源搬移到map中，所以这时的vector是空的。在重新分配空间前，我们通常不应该访问成为移动源的项，但是为了这个实验，我们忽略它：

   ```c++
   $ ./copying_items
   (1, one), (2, two), (3, three),
   (1, one), (2, two), (3, three), (4, four), (5, five),
   (1, ), (2, ), (3, ), (4, ), (5, ),
   ```

## How it works...

 std::copy是STL中最简单的算法之一，其实现也非常短。我们可以看一下其等价的实现：

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

这段代码很朴素，使用for循环将一个容器中的元素一个个的拷贝到另一个容器中。此时，有人就可能会发问："使用for循环的实现非常简单，并且还不用返回值。为什么要在标准库实现这样的算法？"，这是个不错的问题。

std::copy并非能让代码大幅度减少的一个实现，很多其他的算法实现其实非常复杂。这种实现其实在代码层面并不明显，但STL算法更多的在于做了很多底层优化。

