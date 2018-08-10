# 从容器中删除指定元素

复制、转换和过滤是对一段数据常做的操作。本节，我们将重点放在过滤元素上。

将过滤出的元素从数据结构中移除，或是简单的移除其中一个，但对于不同数据结构来说，操作上就完全不一样了。在链表中(比如`std::list`)，只要将对应节点的指针进行变动就好。不过，对于连续存储的结构体来说(比如`std::vector`，`std::array`，还有部分`std::deque`)，删除相应的元素时，将会有其他元素来替代删除元素的位置。当一个元素槽空出来后，那么后面所有的元素都要进行移动，来将这个空槽填满。这个听起来都很麻烦，不过本节中我们只是想要从字符串中移除空格，这个功能没有太多的工作量。

当我们定义了一个结构体时，我们是不会考虑如何将其元素进行删除的。当需要做这件事的时候，我们才会注意到。STL中的`std::remove`和`std::remove_if`函数可以给我们提供帮助。

## How to do it...

我们将通过不同的方式将`vector`中的元素进行删除：

1. 包含必要的头文件，并声明所使用的命名空间。

   ``` c++
   #include <iostream>
   #include <vector>
   #include <algorithm>
   #include <iterator>

   using namespace std;
   ```

2. 一个简单的打印辅助函数，用来打印`vector`中的内容：

   ```c++
   void print(const vector<int> &v)
   {
       copy(begin(v), end(v), ostream_iterator<int>{cout, ", "});
       cout << '\n';
   }
   ```

3. 我们将使用简单的整数对`vector`进行实例化。然后，对`vector`进行打印，这样就能和后面的结果进行对比：

   ```c++
   int main()
   {
       vector<int> v {1, 2, 3, 4, 5, 6};
       print(v);
   ```

4. 现在，我们移除`vector`中所有的2。`std::remove`将2值移动到其他位置，这样这个值相当于消失了。因为`vector`长度在移除元素后变短了，`std::remove`将会返回一个迭代器，这个迭代器指向新的末尾处。旧的`end`迭代器所指向的地方，实际上就没有什么意义了，所以我们可以告诉`vector`将这个位置进行擦除。我们使用两行代码来完成这个任务：

   ```c++
       {
           const auto new_end (remove(begin(v), end(v), 2));
           v.erase(new_end, end(v));
       }
       print(v);
   ```

5. 现在，我们来移除奇数。为了完成移除，我们需要实现一个谓词函数，这个函数用来告诉程序哪些值是奇数，然后结合`std::remove_if`来使用。

   ```c++
       {
           auto odd_number ([](int i) { return i % 2 != 0; });
           const auto new_end (
           	remove_if(begin(v), end(v), odd_number));
           v.erase(new_end, end(v));
       }
       print(v);
   ```

6. 下一个尝试的算法是`std::replace`。我们使用这个函数将所有4替换成123。与`std::replace`函数对应，`std::replace_if`也存在于STL中，同样可以接受谓词函数：

   ```c++
       replace(begin(v), end(v), 4, 123);
       print(v);
   ```

7. 让我们重新初始化`vector`，并为接下来的实验创建两个空的`vector`：

   ```c++
       v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
       
   	vector<int> v2;
       vector<int> v3;
   ```

8. 然后，我们实现两个判读奇偶数的谓词函数：

   ```c++
   	auto odd_number ([](int i) { return i % 2 != 0; });
   	auto even_number ([](int i) { return i % 2 == 0; });
   ```

9. 接下来的两行做的事情完全相同。其将偶数拷贝到v2和v3中。第一行使用`std::remove_copy_if`函数，当相应数值不满足谓词条件时，函数会从源容器中拷贝到另一个容器中。第二行`std::copy_if`则是拷贝满足谓词条件的元素。

   ```c++
       remove_copy_if(begin(v), end(v),
       	back_inserter(v2), odd_number);
       copy_if(begin(v), end(v),
       	back_inserter(v3), even_number); 
   ```

10. 然后，打印这两个`vector`，其内容应该是完全相同的。

    ```c++
        print(v2);
        print(v3);
    }
    ```

11. 编译运行程序。第一行输出的是`vector`初始化的值。第二行是移除2之后的内容。接下来一行是移除所有奇数后的结果。第4行是将4替换为123的结果。最后两行则是v2和v3中的内容：

    ```c++
    $ ./removing_items_from_containers
    1, 2, 3, 4, 5, 6,
    1, 3, 4, 5, 6,
    4, 6,
    123, 6,
    2, 4, 6, 8, 10,
    2, 4, 6, 8, 10,
    ```

## How it works...

这里我们使用了很多与排序算法相关的函数：

| 算法函数                                                     | 作用                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [std::remove](https://zh.cppreference.com/w/cpp/algorithm/remove) | 接受一个容器范围和一个具体的值作为参数，并且移除对应的值。返回一个新的end迭代器，用于修改容器的范围。 |
| [std::replace](https://zh.cppreference.com/w/cpp/algorithm/replace) | 接受一个容器范围和两个值作为参数，将使用第二个数值替换所有与第一个数值相同的值。 |
| [std::remove_copy](https://zh.cppreference.com/w/cpp/algorithm/remove_copy) | 接受一个容器范围，一个输出迭代器和一个值作为参数。并且将所有不满足条件的元素拷贝到输出迭代器的容器中。 |
| [std::replace_copy](https://zh.cppreference.com/w/cpp/algorithm/replace_copy) | 与`std::replace`功能类似，但与`std::remove_copy`更类似些。源容器的范围并没有变化。 |
| [std::copy_if](https://zh.cppreference.com/w/cpp/algorithm/copy) | 与`std::copy`功能相同，可以多接受一个谓词函数作为是否进行拷贝的依据。 |

> Note:
>
> 表中没有if的算法函数，都有一个*_if版本存在，其能接受谓词函数，通过谓词函数判断的结果来进行相应的操作。

