# 标准算法的自动并行

C++17对并行化的一个重要的扩展，就是对标准函数的执行策略进行了修改。69个标准算法都能并行到不同的核上运行，甚至是向量化。

对于使用者来说，如果经常使用STL中的算法，那么就能很轻易的进行并行。可以通过基于现存的STL算法一个执行策略，然后就能享受并行带来的好处。

本节中，我们将实现一个简单的程序(通过一个不太严谨的使用场景)，其中使用了多个STL算法。使用这些算法时，我们将看到如何在C++17中，使用执行策略让这些算法并行化。本节最后一个子节中，我们会了解不同执行策略的区别。

## How to do it...

本节，将使用标准算法来完成一个程序。这个程序本身就是在模拟我们实际工作中的场景。当使用这些标准算法时，我们为了得到更快的性能，将执行策略嵌入其中：

1. 包含必要的头文件，并声明所使用的命名空间。其中`execution`头文件是C++17之后加入的：

   ```c++
   #include <iostream>
   #include <vector>
   #include <random>
   #include <algorithm>
   #include <execution>
   
   using namespace std;
   ```

2. 这里声明一个谓词函数，其用来判断给定数值的奇偶：

   ```c++
   static bool odd(int n) { return n % 2; }
   ```

3. 主函数中先来定义一个很大的`vector`。我们将对其进行填充，并对其中数值进行计算。这个代码的执行速度是非常非常慢的。对于不同配置的电脑来说，这个`vector`的尺寸可能会有变化：

   ```c++
   int main()
   {
   	vector<int> d (50000000);
   ```

4. 为了向`vector`中塞入随机值，我们对随机数生成器进行了实例化，并选择了一种分布进行生成，并且将其打包成为一个可调用的对象。如果你对随机数生成器不太熟，那么你可以回看一下本书的第8章：

   ```c++
   	mt19937 gen;
   
   	uniform_int_distribution<int> dis(0, 100000);
   	auto rand_num ([=] () mutable { return dis(gen); }); 
   ```

5. 现在，`std::generate`算法会用随机值将`vector`填满。这个算法是C++17新加入的算法，其能接受一种新的参数——执行策略。我们在这个位置上填入`std::execution::par`，其能让代码进行自动化并行。通过这个参数的传入，可以使用多线程的方式对`vector`进行填充，如果我们的电脑有多核CPU，那么就可以大大节约我们的时间：

   ```c++
   	generate(execution::par, begin(d), end(d), rand_num);
   ```

6. `std::sort`想必大家都是非常熟悉了。C++17对其也提供了执行策略的参数：

   ```c++
   	sort(execution::par, begin(d), end(d));
   ```

7. 还有`std::reverse`:

   ```c++
   	reverse(execution::par, begin(d), end(d));
   ```

8. 然后，我们使用`std::count_if`来计算`vector`中奇数的个数。并且也可以通过添加执行策略参数对该算法进行加速：

   ```c++
   	auto odds (count_if(execution::par, begin(d), end(d), odd));
   ```

9. 最后，将结果进行打印：

   ```c++
   	cout << (100.0 * odds / d.size())
   		<< "% of the numbers are odd.\n";
   }
   ```

10. 编译并运行程序，就能得到下面的输出。整个程序中我们就使用了一种执行策略，我们对不同执行策略之间的差异也是非常感兴趣。这个就留给读者当做作业。去了解一下不同的执行策略，比如`seq`，`par`和`par_vec`。 不过，对于不同的执行策略，我们肯定会得到不同的执行时间：

    ```c++
    $ ./auto_parallel
    50.4% of the numbers are odd.
    ```

## How it works...

本节并没有设计特别复杂的使用场景，这样就能让我们集中精力与标准库函数的调用上。并行版本的算法和标准串行的算法并没有什么区别。其差别就是多了一个参数，也就是执行策略。

让我们结合以下代码，来看三个核心问题：

```c++
generate(execution::par, begin(d), end(d), rand_num);
sort( execution::par, begin(d), end(d));
reverse( execution::par, begin(d), end(d));

auto odds (count_if(execution::par, begin(d), end(d), odd));
```

**哪些STL可以使用这种方式进行并行？**

69种存在的STL算法在C++17标准中，都可以使用这种方式进行并行，还有7种新算法也支持并行。虽然这种升级对于很多实现来说很伤，但是也只是在接口上增加了一个参数——执行策略参数。这也不是意味着我们总要提供一个执行策略参数。并且执行策略参数放在了第一个参数的位置上。

这里有69个升级了的算法。并且有7个新算法在一开始就支持了并发：

```c++
adjacent difference, adjacent find.
all_of, any_of, none_of
copy
count
equal
fill
find
generate
includes
inner product
in place merge, merge
is heap, is partitioned, is sorted
lexicographical_compare
min element, minmax element
mismatch
move
n-th element
partial sort, sort copy
partition
remove + variations
replace + variations
reverse / rotate
search
set difference / intersection / union /symmetric difference
sort
stable partition
swap ranges
transform
unique
```

详细的内容可以查看[C++ Reference](http://en.cppreference.com/w/cpp/experimental/parallelism/existing)。([参考页面](https://www.bfilipek.com/2017/08/cpp17-details-parallel.html))

这些算法的升级是一件令人振奋的事！如果我们之前的程序使用了很多的STL算法，那么就很容易的将这些算法进行并行。这里需要注意的是，这样的的改变并不意味着每个程序自动化运行N次都会很快，因为多核编程更为复杂，所要注意的事情更多。

不过，在这之前我们现在都会用`std::thread`，`std::async`或是第三方库进行复杂的并行算法设计，而现在我们可以以更加优雅、与操作系统不相关的方式进行算法的并行化。

**执行策略是如何工作的？**

执行策略会告诉我们的标准函数，以何种方式进行自动化并行。

`std::execution`命名空间下面，有三种策略类型：

| 策略                                                         | 含义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [sequenced_policy](https://zh.cppreference.com/w/cpp/algorithm/execution_policy_tag_t) | 算法使用串行的方式执行，这与原始执行方式没有什么区别。全局可用的实例命名为`std::execution::seq`。 |
| [parallel_policy](https://zh.cppreference.com/w/cpp/algorithm/execution_policy_tag_t) | 算法使用多线程的方式进行执行。全局可用的实例命名为`std::execution::par`。 |
| [parallel_unsequenced_policy](https://zh.cppreference.com/w/cpp/algorithm/execution_policy_tag_t) | 算法使用多线程的方式进行执行。并允许对代码进行向量化。在这个例子中，线程间可以对内存进行交叉访问，向量化的内容可以在同一个线程中执行。全局可用的实例命名为`std::execution::par_unseq`。 |

执行策略意味着我们需要进行严格限制。严格的约定，让我们有更多并行策略可以使用：

- 并行算法对所有元素的访问，必须不能导致死锁或数据竞争。
- 向量化和并行化中，所有可访问的函数不能使用任何一种阻塞式同步。

我们需要遵守这些规则，这样才不会将错误引入到程序中。

> Note：
>
> STL的自动并行化，并总能保证有加速。因为具体的情况都不一样，所以可能在很多情况下并行化并没有加速。多核编程还是很有难度的。

**向量化是什么意思？**

向量化的特性需要编译器和CPU都支持，让我们先来简单的了解一下向量化是如何工作的。假设我们有一个非常大的`vector`。简单的实现可以写成如下的方式：

```c++
std::vector<int> v {1, 2, 3, 4, 5, 6, 7 /*...*/};

int sum {std::accumulate(v.begin(), v.end(), 0)};
```

编译器将会生成一个对`accumulate`调用的循环，其可能与下面代码类似：

```c++
int sum {0};

for (size_t i {0}; i < v.size(); ++i) {
	sum += v[i];
}
```

从这点说起，当编译器开启向量化时，就会生成类似如下的代码。每次循环会进行4次累加，这样循环次数就要比之前减少4倍。为了简单说明问题，我们这里没有考虑不为4倍数个元素的情况：

```c++
int sum {0};
for (size_t i {0}; i < v.size() / 4; i += 4) {
	sum += v[i] + v[i+1] + v[i + 2] + v[i + 3];
}
// if v.size() / 4 has a remainder,
// real code has to deal with that also.
```

为什么要这样做呢？很多CPU指令都能支持这种操作`sum += v[i] + v[i+1] + v[i+2] + v[i+3]；`，只需要一个指令就能完成。使用尽可能少的指令完成尽可能多的操作，这样就能加速程序的运行。

自动向量化非常困难，因为编译器需非常了解我们的程序，这样才能进行加速的情况下，不让程序的结果出错。目前，至少可以通过使用标准算法来帮助编译器。因为这样能让编译器更加了解哪些数据流能够并行，而不是从复杂的循环中对数据流的依赖进行分析。



