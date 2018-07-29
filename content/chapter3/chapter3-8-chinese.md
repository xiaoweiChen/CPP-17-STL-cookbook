# 构建zip迭代适配器

不同的编程语言引领了不同的编程方式。不同语言有各自的受众群体，因为表达方式的不同，所以对于优雅地定义也不同。

纯函数式编程算是编程风格中一种比较特别的方式。其与C和C++命令方式编程的风格大相径庭。虽然风格迥异，但是纯函数式编程却能在大多数情况下产生非常优雅地代码。

这里用向量点乘为例，使用函数式方法优雅地实现这个功能。给定两个向量，然后让对应位置上的两个数字相乘，然后将所有数字加在一起。也就是`(a, b, c) * (d, e, f)`的结果为`(a * e + b * e + c * f) `。我们在C和C++也能完成这样的操作。代码可能类似如下的方式：

```c++
std::vector<double> a {1.0, 2.0, 3.0};
std::vector<double> b {4.0, 5.0, 6.0};
double sum {0};
for (size_t i {0}; i < a.size(); ++i) {
	sum += a[i] * b[i];
}
// sum = 32.0
```

如何使用其他语言让这段代码更优雅呢？

Haskell是一种纯函数式语言，其使用一行代码就能计算两个向量的点积：

![](../../images/chapter3/3-8-1.png)

Python虽然不是纯函数式编程语言，但是也会提供类似功能：

![](../../images/chapter3/3-8-2.png)

STL提供了相应的函数实现`std::inner_product`，也能在一行之内完成向量点积。不过，其他语言中在没有相应的库对某种操作进行支持的情况下，也能做到在一行之内完成。

不需要对两种语言的语法进行详细了解的情况下，大家都应该能看的出，两个例子中最重要的就是zip函数。这个函数做了什么？假设我们有两个向量a和b，变换后将两个向量混合在一起。例如：`[a1, a2, a3]`和`[b1, b2, b3]`，使用zip函数处理的结果为`[(a1, b1), (a2, b2), (a3, b3)]`。让我们仔细观察这个例子，就是将两个向量连接在了一起。

现在，关联的数字可以直接进行加法，然后累加在一起。在Haskell和Python的例子中我们看到，这个过程不需要任何循环或索引变量。[译者注：Python中是有循环的……]

这里没法让C++代码如同Haskell或Python那样优雅和通用，不过本节的内容就是为了实现一个类似的迭代器——zip迭代器——然后使用这个迭代器。向量点积有特定的库支持，至于是哪些库，以及这些库如何使用，并不在本书的描述范围内。不过，本节的内容将尝试展示一种基于迭代器的方式，来帮助你使用通用的模块另外完成编程。

## How to do it...

本节中，我们会实现一个类似Haskell和Python中的zip函数。为了不对迭代器的机制产生影响，`vector`中的变量这里写死为`double`：

1. 包含头文件

   ```c++
   #include <iostream>
   #include <vector>
   #include <numeric>
   ```

2. 定义`zip_iterator`类。同时也要实现一个范围类`zip_iterator`，这样我们在每次迭代时就能获得两个值。这也意味着我们同时遍历两个迭代器：

   ```c++
   class zip_iterator {
   ```

3. zip迭代器的容器中需要保存两个迭代器：

   ```c++
   	using it_type = std::vector<double>::iterator;

   	it_type it1;
   	it_type it2;
   ```

4. 构造函数会将传入的两个容器的迭代器进行保存，以便进行迭代：

   ```c++
   public:
       zip_iterator(it_type iterator1, it_type iterator2)
       	: it1{iterator1}, it2{iterator2}
       {}
   ```

5. 增加zip迭代器就意味着增加两个成员迭代器：

   ```c++
       zip_iterator& operator++() {
           ++it1;
           ++it2;
           return *this;
       }
   ```

6. 如果zip中的两个迭代器来自不同的容器，那么他们一定不相等。通常，这里会用逻辑或(||)替换逻辑与(&&)，但是这里我们需要考虑两个容器长度不一样的情况。这样的话，我们需要在比较的时候同时匹配两个容器。这样，我们就能遍历完其中一个容器时，及时停下循环：

   ```c++
   	bool operator!=(const zip_iterator& o) const {
       	return it1 != o.it1 && it2 != o.it2;
       }
   ```

7. 逻辑等操作符可以使用逻辑不等的操作符的实现，是需要将结果取反即可：

   ```c++
       bool operator==(const zip_iterator& o) const {
       	return !operator!=(o);
       }
   ```

8. 解引用操作符用来访问两个迭代器指向的值：

   ```c++
       std::pair<double, double> operator*() const {
       	return {*it1, *it2};
       }
   };
   ```

9. 迭代器算是实现完了。我们需要让迭代器兼容STL算法，所以我们对标准模板进行了特化。这里讲迭代器定义为一个前向迭代器，并且解引用后返回的是一对`double`值。虽然，本节我们没有使用`difference_type`，但是对于不同编译器实现的STL可能就需要这个类型：

   ```c++
   namespace std {
   template <>
   struct iterator_traits<zip_iterator> {
       using iterator_category = std::forward_iterator_tag;
       using value_type = std::pair<double, double>;
       using difference_type = long int;
   };
   }
   ```

10. 现在来定义范围类，其`begin`和`end`函数返回`zip`迭代器：

    ```c++
    class zipper {
        using vec_type = std::vector<double>;
        vec_type &vec1;
        vec_type &vec2; 
    ```

11. 这里需要从zip迭代器中解引用两个容器中的值：

    ```c++
    public:
        zipper(vec_type &va, vec_type &vb)
        	: vec1{va}, vec2{vb}
        {}
    ```

12. `begin`和`end`函数将返回指向两容器开始的位置和结束位置的迭代器对：

    ```c++
        zip_iterator begin() const {
        	return {std::begin(vec1), std::begin(vec2)};
        }
        zip_iterator end() const {
       		return {std::end(vec1), std::end(vec2)};
        }
    };
    ```

13. 如Haskell和Python的例子一样，我们定义了两个`double`为内置类型的`vector`。这里我们也声明了所使用的命名空间。

    ```c++
    int main()
    {
        using namespace std;
        vector<double> a {1.0, 2.0, 3.0};
        vector<double> b {4.0, 5.0, 6.0};
    ```

14. 可以直接使用两个`vector`对`zipper`类进行构造：

    ```c++
    	zipper zipped {a, b};
    ```

15. 我们将使用`std::accumulate`将所有值累加在一起。这里我们不能直接对`std::pair<double, double>`实例的结果进行累加，因为这里没有定义`sum`变量。因此，我们需要定义一个辅助Lambda函数来对这个组对进行操作，将两个数相乘，然后进行累加。Lambda函数指针可以作为`std::accumulate`的一个参数传入：

    ```c++
        const auto add_product ([](double sum, const auto &p) {
        	return sum + p.first * p.second;
        });
    ```

16. 现在，让我们来调用`std::accumulate`将所有点积的值累加起来：

    ```c++
        const auto dot_product (accumulate(
        	begin(zipped), end(zipped), 0.0, add_product));
    ```

17. 最后，让我们来打印结果：

    ```c++
    	cout << dot_product << '\n';
    }
    ```

18. 编译运行后，得到正确的结果：

    ```c++
    32
    ```

## There's more...

OK，这里使用了语法糖来完成了大量的工作，不过这和Haskell的例子也相差很远，还不够优雅。我们的设计中有个很大的缺陷，那就是只能处理`double`类型的数据。通过模板代码和特化类，`zipper`类会变得更通用。这样，我们就能将`list`和`vector`或`deque`和`map`这样不相关的容器合并起来。

为了让设计的类更加通用，其中设计的过程是不容忽视的。幸运的是，这样的库已经存在。Boost作为STL库的先锋，已经支持了`zip_iterator`。这个迭代器非常简单、通用。

顺便提一下，如果你想看到了使用C++实现的更优雅的点积，并且不关心`zip`迭代器相关的内容，那么你可以了解一下`std::valarray`。例子如下，自己看下：

```c++
#include <iostream>
#include <valarray>
int main()
{
    std::valarray<double> a {1.0, 2.0, 3.0};
    std::valarray<double> b {4.0, 5.0, 6.0};
    std::cout << (a * b).sum() << '\n';
}
```

**范围库**

这是C++中非常有趣的一个库，其支持`zipper`和所有迭代适配器、滤波器等等。其受到Boost范围库的启发，并且某段时间内里，很有可能进入C++17标准。不幸的是，我们只能在下个标准中期待这个特性的加入。这种性能可以带来更多的便利，能让我们想表达的东西通过C++快速实现，并可以通过将通用和简单的模块进行组合，来表现比较复杂的表达式。

在文档中对其描述中，有个非常简单的例子：

1. 计算从1到10数值的平方：

   ```c++
   const int sum = accumulate(view::ints(1)
                           | view::transform([](int i){return i*i;})
                           | view::take(10), 0); 
   ```

2. 从数值`vector`中过滤出非偶数数字，并且将剩下的数字转换成字符串：

   ```c++
   std::vector<int> v {1,2,3,4,5,6,7,8,9,10};

   auto rng = v | view::remove_if([](int i){return i % 2 == 1;})
   		    | view::transform([](int i){return std::to_string(i);});
   // rng == {"2"s,"4"s,"6"s,"8"s,"10"s};
   ```

如果你等不及想要了解这些有趣的特性，可以看一下范围类的文档，https://ericniebler.github.io/range-v3 。

