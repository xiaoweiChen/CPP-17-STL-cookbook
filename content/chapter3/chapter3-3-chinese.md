# 使用迭代适配器填充通用数据结构

大多数情况下，我们想要用数据来填充任何容器，不过数据源和容器却没有通用的接口。这种情况下，我们就需要人工的去编写算法，将相应的数据推入容器中。不过，这会分散我们解决问题的注意力。

不同数据结构间的数据传递现在可以只通过一行代码就完成，这要感谢STL中的**迭代适配器**。本节会展示如何使用迭代适配器。

## How to do it...

本节中，我们使用一些迭代器包装器，展示如何使用包装器，并了解其如何在编程任务中给予我们帮助。

1. 包含必要的头文件。

   ```c++
   #include <iostream>
   #include <string>
   #include <iterator>
   #include <sstream>
   #include <deque>
   ```

2. 声明使用的命名空间。

   ```c++
   using namespace std;
   ```

3. 开始使用`std::istream_iterator`。这里我们特化为`int`类型。这样，迭代器就能将标准输入解析成整数。例如，当我们遍历这个迭代器，其就和`std::vector<int>`一样了。`end`迭代器的类型没有变化，但不需要构造参数:

   ```c++ 
   int main()
   {
       istream_iterator<int> it_cin {cin};
       istream_iterator<int> end_cin;
   ```

4. 接下来，我们实例化一个`std::deque<int>`，并且将标准输入中的所有数字拷贝到队列中。队列本身不是一个迭代器，所以我们使用`std::back_inserter`辅助函数将队列包装入`std::back_insert_iterator`中。这样指定的迭代器就能执行`v.pack_back(item)`，将标准输入中的每个元素放入容器中。这样就能让队列自动增长。

   ```c++
       deque<int> v;
       copy(it_cin, end_cin, back_inserter(v));	
   ```

5. 接下来，我们使用`std::istringstream`将元素拷贝到队列中部。先使用字符串，来定义一个字符流的实例：

   ```c++
   	istringstream sstr {"123 456 789"};
   ```

6. 我们需要选择列表的插入点。这个点必须在中间，我们使用队列的起始指针，然后使用`std::next`函数将其指向中间位置。函数第二个参数的意思就是让指针前进多少，这里选择`v.size() / 2`步，也就是队列的正中间位置(这里我们将`v.size()`强转为`int`类型，因为`std::next`第二个参数类型为`difference_type`，是和第一个迭代器参数间的距离。因此，该类型是个有符号类型。根据编译选项，如果我们不进行显式强制转化，编译器可能会报出警告)。

   ```c++
       auto deque_middle (next(begin(v),
       	 static_cast<int>(v.size()) / 2));
   ```

7. 现在，我们可以从输入流中一步步的拷贝整数到队列当中。另外，流的`end`包装迭代器为空的` std::istream_iterator<int> `。这个队列已经被包装到一个插入包装器中，也就是成为`std::insert_iterator`的一个实例，其指向队列中间位置的迭代器，我们用`deque_middle`表示:

   ```c++
   	copy(istream_iterator<int>{sstr}, {}, inserter(v, deque_middle));
   ```

8. 现在，让我们使用`std::front_insert_iterator`插入一些元素到队列中部：

   ```c++
       initializer_list<int> il2 {-1, -2, -3};
       copy(begin(il2), end(il2), front_inserter(v));
   ```

9. 最后一步将队列中的全部内容打印出来。`std::ostream_iterator`作为输出迭代器，在我们的例子中其就是从`std::cout`拷贝打印出的信息，并将各个元素使用逗号隔开：

   ```c++
       copy(begin(v), end(v), ostream_iterator<int>{cout, ", "});
       cout << '\n';
   }
   ```

10. 编译并运行，即有如下的输出。你能找到那些数字是由哪行的代码插入的吗？

    ```c++
    $ echo "1 2 3 4 5" | ./main
    -3, -2, -1, 1, 2, 123, 456, 789, 3, 4, 5,
    ```

## How it works...

本节我们使用了很多不同类型的迭代适配器。他们有一点是共同的，会将一个对象包装成迭代器。

**std::back_insert_iterator**

`back_insert_iterator`可以包装`std::vector`、`std::deque`、`std::list`等容器。其会调用容器的`push_back`方法在容器最后插入相应的元素。如果容器实例不够长，那么容器的容量会自动增长。

**std::front_insert_iterator**

`front_insert_iterator`和`back_insert_iterator`一样，不过`front_insert_iterator`调用的是容器的`push_front`函数，也就是在所有元素前插入元素。这里需要注意的是，当对类似于`std::vector`的容器进行插入时，其已经存在的所有元素都要后移，从而空出位置来放插入元素，这会对性能造成一定程度的影响。

**std::insert_iterator**

这个适配器与其他插入适配器类似，不过能在容器的中间位置插入新元素。使用`std::inserter`包装辅助函数需要两个参数。第一个参数是容器的实例，第二个参数是迭代器指向的位置，就是新元素插入的位置。

**std::istream_iterator**

`istream_iterator`是另一种十分方便的适配器。其能对任何`std::istream`使用(文件流或标准输入流)，并且可以根据实例的具体特化类型，对流进行分析。本节中，我们使用了`std::istram_iterator<int>(std::cin)`，其会将整数从标准输入中拉出来。

通常，对于流来说，其长度我们是不知道的。这就存在一个问题，也就是`end`迭代器指向的位置在哪里？对于流迭代器来说，它就知道相应的`end`迭代器的位置。这样就使得迭代器的比较更加高效，不需要通过遍历来完成。这样就是为什么`end`流迭代器不需要传入任何参数的原因。

**std::ostream_iterator**

`ostream_iterator`和`istream_iterator`类似，不过是用来进行输出的流迭代器。与`istream_iterator`不同在于，构造时需要传入两个参数，且第二个参数必须要是一个字符串，这个字符串将会在各个元素之后，推入输出流中。这样我们就能很容易的在元素中间插入逗号或者换行的符号，以便用户进行观察。