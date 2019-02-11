# 保持对std::vector实例的排序

`array`和`vector`不会对他们所承载的对象进行排序。有时我们去需要排序，但这不代表着我们总是要去切换数据结构，需要排序能够自动完成。在我们的例子有如有一个`std::vector`实例，将添加元素后的实例依旧保持排序，会是一项十分有用的功能。

## How to do it...

本节中我们使用随机单词对`std::vector`进行填充，然后对它进行排序。并在插入更多的单词的同时，保证`vector`实例中单词的整体排序。

1. 先包含必要的头文件。

   ```c++
   #include <iostream>
   #include <vector>
   #include <string>
   #include <algorithm>
   #include <iterator>
   #include <cassert>
   ```

2. 声明所要使用的命名空间。

   ```c++
   using namespace std;
   ```

3. 完成主函数，使用一些随机单词填充`vector`实例。

   ```c++
   int main()
   {
       vector<string> v {"some", "random", "words",
                         "without", "order", "aaa",
                         "yyy"};
   ```

4. 对vector实例进行排序。我们使用一些断言语句和STL中自带的`is_sorted`函数对是否排序进行检查。

   ```c++
       assert(false == is_sorted(begin(v), end(v)));
       sort(begin(v), end(v));
       assert(true == is_sorted(begin(v), end(v)));
   ```

5. 这里我们使用`insert_sorted`函数添加随机单词到已排序的`vector`中，这个函数我们会在后面实现。这些新插入的单词应该在正确的位置上，并且`vector`实例需要保持已排序的状态。

   ```c++
       insert_sorted(v, "foobar");
       insert_sorted(v, "zzz");
   ```

6. 现在，我们来实现`insert_sorted`函数。

   ```c++
   void insert_sorted(vector<string> &v, const string &word)
   {
       const auto insert_pos (lower_bound(begin(v), end(v), word));
       v.insert(insert_pos, word);
   }
   ```

7. 回到主函数中，我们将`vector`实例中的元素进行打印。

   ```c++
       for (const auto &w : v) {
       	cout << w << " ";
       }
       cout << '\n';
   }	
   ```

8. 编译并运行后，我们得到如下已排序的输出。

   ```c++
   aaa foobar order random some without words yyy zzz
   ```

## How it works...

程序整个过程都是围绕`insert_sorted`展开，这也就是本节所要说明的：对于任意的新字符串，通过计算其所在位置，然后进行插入，从而保证`vector`整体的排序性。不过，这里我们假设的情况是，在插入之前，`vector`已经排序。否则，这种方法无法工作。

这里我们使用STL中的`lower_bound`对新单词进行定位，其可接收三个参数。头两个参数是容器开始和结尾的迭代器。这确定了我们单词`vector`的范围。第三个参数是一个单词，也就是要被插入的那个。函数将会找到大于或等于第三个参数的首个位置，然后返回指向这个位置的迭代器。

获取了正确的位置，那就使用`vector`的成员函数`insert`将对应的单词插入到正确的位置上。

## There's more...

`insert_sorted`函数很通用。如果需要其适应不同类型的参数，这样改函数就能处理其他容器所承载的类型，甚至是容器的类似，比如`std::set`、`std::deque`、`std::list`等等。(这里需要注意的是成员函数`lower_bound`与 `std::lower_bound`等价，不过成员函数的方式会更加高效，因为其只用于对应的数据集合)

```c++
template <typename C, typename T>
void insert_sorted(C &v, const T &item)
{
    const auto insert_pos (lower_bound(begin(v), end(v), item));
    v.insert(insert_pos, item);
}
```

当我们要将`std::vector`类型转换为其他类型时，需要注意的是并不是所有容器都支持`std::sort`。该函数所对应的算法需要容器为可随机访问容器，例如`std::list`就无法进行排序。