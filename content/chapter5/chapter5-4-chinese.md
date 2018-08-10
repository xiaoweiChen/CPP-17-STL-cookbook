# 改变容器内容

如果说`std::copy`是STL中最简单的算法，那么`std::transform`就是第二简单的算法。和`copy`类似，其可将容器某一范围的元素放置到其他容器中，在这个过程中允许进行一些变换(变换函数会对输入值进行一定操作，然后再赋给目标容器)。此外，两个具有不同元素类型的容间也可以使用这个函数。这个函数超级简单，并且非常有用，这个函数会让标准组件具有更好的可移植性。

## How to do it...

本节，我们将使用`std::transform`在拷贝的同时，修改`vector`中的元素：

1. 包含必要的头文件，并且声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <vector>
   #include <string>
   #include <sstream>
   #include <algorithm>
   #include <iterator>

   using namespace std;
   ```

2. `vector`由简单的整数组成：

   ```c++
   int main()
   {
   	vector<int> v {1, 2, 3, 4, 5};
   ```

3. 为了打印元素，会将所有元拷贝素到`ostream_iterator`适配器中。`transform`函数可以接受一个函数对象，其能在拷贝过程中对每个元素进行操作。这个例子中，我们将计算每个元素的平方值，所以代码将打印出平方数。因为直接进行了打印，所以平方数并没有进行保存：

   ```c++
       transform(begin(v), end(v),
           ostream_iterator<int>{cout, ", "},
           [] (int i) { return i * i; });
       cout << '\n';
   ```

4. 再来做另一个变换。例如，对于数字3来说，显示成`3^2 = 9`显然有更好的可读性。下面的辅助函数`int_to_string`函数对象就会使用`std::stringstream`对象进行打印操作：

   ```c++
   auto int_to_string ([](int i) {
       stringstream ss;
       ss << i << "^2 = " << i * i;
       return ss.str();
   });
   ```

5. 这样就可以将整型值放入字符串中。可以说我么将这个证书映射到字符串中。使用`transform`函数，使我们可以拷贝所有数值到一个字符串`vector`中：

   ```c++
       vector<string> vs;

       transform(begin(v), end(v), back_inserter(vs),
       	int_to_string);
   ```

6. 在打印完成后，我们的例子就结束了：

   ```c++
       copy(begin(vs), end(vs),
      		ostream_iterator<string>{cout, "\n"});
   }
   ```

7. 编译并运行程序：

   ```c++
   $ ./transforming_items_in_containers
   1, 4, 9, 16, 25,
   1^2 = 1
   2^2 = 4
   3^2 = 9
   4^2 = 16
   5^2 = 25
   ```

## How it works...

`std::transform`函数工作原理和`std::copy`差不多，不过在拷贝的过程中会对源容器中的元素进行变换，这个变换函数由用户提供。