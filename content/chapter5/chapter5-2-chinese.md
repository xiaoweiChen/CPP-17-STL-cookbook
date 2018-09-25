# 容器元素排序

排序是一项很常见的任务，并且可以通过各种各样的方式进行。每个计算机科学专业的学生，都学过很多排序算法(包括这些算法的性能和稳定性)。

因为这是个已解决的问题，所以开发者没必要浪费时间再次来解决排序问题，除非是出于学习的目的。

## How to do it...

本节中，我们将展示如何使用`std::sort`和`std::partial_sort`。

1. 首先，包含必要的头文件和声明所使用的命名空间。

   ```c++
   #include <iostream>
   #include <algorithm>
   #include <vector>
   #include <iterator>
   #include <random>

   using namespace std;
   ```

2. 我们将打印整数在`vector`出现的次数，为了缩短任务代码的长度，我们在这里写一个辅助函数：

   ```c++
   static void print(const vector<int> &v)
   {
       copy(begin(v), end(v), ostream_iterator<int>{cout, ", "});
       cout << '\n';
   }
   ```

3. 我们开始实例化一个`vector`：

   ```c++
   int main()
   {
   	vector<int> v {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
   ```

4. 因为我们将使用不同的排序函数将`vector`多次打乱，所以我们需要一个随机数生成器：

   ```c++
   	random_device rd;
   	mt19937 g {rd()};
   ```

5. `std::is_sorted`函数会告诉我们，容器内部的值是否已经经过排序。所以这行将打印到屏幕上：

   ```c++
   	cout << is_sorted(begin(v), end(v)) << '\n';
   ```

6. `std::shuffle`将打乱`vector`中的内容，之后我们会再次对`vector`进行排序。前两个参数是容器的首尾迭代器，第三个参数是一个随机数生成器：

   ```c++
   	shuffle(begin(v), end(v), g);
   ```

7. 现在`is_sorted`函数将返回false，所以0将打印在屏幕上，`vector`的元素总量和具体数值都没有变，不过顺序发生了变化。我们会将函数的返回值再次打印在屏幕上：

   ```c++
   	cout << is_sorted(begin(v), end(v)) << '\n';
   	print(v);
   ```

8. 现在，在通过`std::sort`对`vector`进行排序。然后打印是否排序的结果：

   ```c++
   	sort(begin(v), end(v));
       
   	cout << is_sorted(begin(v), end(v)) << '\n';
       print(v);
   ```

9. 另一个比较有趣的函数是`std::partition`。有时候，并不需要对列表完全进行排序，只需要比它前面的某些值小就可以。所以，让使用`partition`将数值小于5的元素排到前面，并打印它们：

   ```c++
       shuffle(begin(v), end(v), g);
       
   	partition(begin(v), end(v), [] (int i) { return i < 5; });
       
   	print(v); 
   ```

10. 下一个与排序相关的函数是`std::partial_sort`。我们可以使用这个函数对容器的内容进行排序，不过只是在某种程度上的排序。其会将`vector`中最小的N个数，放在容器的前半部分。其余的留在`vector`的后半部分，不进行排序：

    ```c++
        shuffle(begin(v), end(v), g);
        auto middle (next(begin(v), int(v.size()) / 2));
        partial_sort(begin(v), middle, end(v));
        
    	print(v);
    ```

11. 当我们要对没做比较操作符的结构体进行比较，该怎么办呢？让我们来定义一个结构体，然后用这个结构体来实例化一个`vector`：

    ```c++
        struct mystruct {
        int a;
        int b;
        };

        vector<mystruct> mv { {5, 100}, {1, 50}, {-123, 1000},
        				   {3, 70}, {-10, 20} };
    ```

12. `std::sort`函数可以将比较函数作为第三个参数进行传入。让我们来使用它，并且传递一个比较函数。为了展示其实如何工作的，我们会对其第二个成员b进行比较。这样，我们将按`mystruct::b`的顺序进行排序，而非`mystruct::a`的顺序：

    ```c++
        sort(begin(mv), end(mv),
        [] (const mystruct &lhs, const mystruct &rhs) {
            return lhs.b < rhs.b;
        });
    ```

13. 最后一步则是打印已经排序的`vector`：

    ```c++
        for (const auto &[a, b] : mv) {
        	cout << "{" << a << ", " << b << "} ";
        }
        cout << '\n';
    }
    ```

14. 编译运行程序。第一个1是由`std::is_sorted`所返回的。之后将`vector`进行打乱后，`is_sorted`就返回0。第三行是打乱后的`vector`。下一个1是使用sort之后进行打印的。然后，`vector`会被再次打乱，并且使用`std::partition`对部分元素进行排序。我们可以看到所有比5小的元素都在左边，比5大的都在右边。我们暂且将现在的顺序认为是乱序。倒数第二行展示了`std::partial_sort`的结果。前半部分的内容进行了严格的排序，而后半部分则没有。最后一样，我们将打印`mystruct`实例的结果。其结果是严格根据第二个成员变量的值进行排序的：

    ```c++
    $ ./sorting_containers
    1
    0
    7, 1, 4, 6, 8, 9, 5, 2, 3, 10,
    1
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
    1, 2, 4, 3, 5, 7, 8, 10, 9, 6,
    1, 2, 3, 4, 5, 9, 8, 10, 7, 6,
    {-10, 20} {1, 50} {3, 70} {5, 100} {-123, 1000}
    ```

## How it works...

这里我们使用了很多与排序算法相关的函数：

| 算法函数                                                     | 作用                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [std::sort](https://zh.cppreference.com/w/cpp/algorithm/sort) | 接受一定范围的元素，并对元素进行排序。                       |
| [std::is_sorted](https://zh.cppreference.com/w/cpp/algorithm/is_sorted) | 接受一定范围的元素，并判断该范围的元素是否经过排序。         |
| [std::shuffle](https://zh.cppreference.com/w/cpp/algorithm/random_shuffle) | 类似于反排序函数；其接受一定范围的元素，并打乱这些元素。     |
| [std::partial_sort](https://zh.cppreference.com/w/cpp/algorithm/partial_sort) | 接受一定范围的元素和另一个迭代器，前两个参数决定排序的范围，后两个参数决定不排序的范围。 |
| [std::partition](https://zh.cppreference.com/w/cpp/algorithm/partition) | 能够接受谓词函数。所有元素都会在谓词函数返回true时，被移动到范围的前端。剩下的将放在范围的后方。 |

对于没有实现比较操作符的对象来说，想要排序就需要提供一个自定义的比较函数。其签名为`bool function_name(const T &lhs, const T &rhs)`，并且在执行过程中无副作用。

当然排序还有其他类似`std::stable_sort`的函数，其能保证排序后元素的原始顺序，`std::stable_partition`也有类似的功能。

> Note:
>
> `std::sort`对于排序有不同的实现。根据所提供的迭代器参数，其实现分为选择排序、插入排序、合并排序，对于元素数量较少的容器可以完全进行优化。在使用者的角度，我们通常都不需要了解这些。