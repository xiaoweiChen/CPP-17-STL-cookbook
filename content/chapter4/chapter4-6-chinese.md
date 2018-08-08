# 使用std::accumulate和Lambda函数实现transform_if

大多数用过`std::copy_if`和`std::transform`的开发者可能曾经疑惑过，为什么标准库里面没有`std::transform_if`。`std::copy_if`会将源范围内符合谓词判断的元素挑出来，不符合条件的元素忽略。而`std::transform`会无条件的将源范围内所有元素进行变换，然后放到目标范围内。这里的变换谓词是由用户提供的一个函数，这个函数不会太复杂，比如乘以多个数或将元素完全变换成另一种类型。

这两个函数很早就存在了，不过到现在还是没有`std::transform_if`函数。本节就来实现这个函数。看起来实现这个函数并不难，可以通过谓词将对应的元素选择出来，然后将这些挑选出来的元素进行变换。不过，我们会利用这个机会更加深入的了解Lambda表达式。

## How to do it...

我们将来实现我们的`transform_if`函数，其将会和`std::accumulate`一起工作。

1. 包含必要的头文件。

   ```c++
   #include <iostream>
   #include <iterator>
   #include <numeric>
   ```

2. 首先，我们来实现一个`map`函数。其能接受一个转换函数作为参数，然后返回一个函数对象，这个函数对象将会和`std::accumulate`一起工作。

   ```c++
   template <typename T>
   auto map(T fn)
   {
   ```

3. 当传入一个递减函数时，我们会返回一个函数对象，当这个函数对象调用递减函数时，其会返回另一个函数对象，这个函数对象可以接受一个累加器和一个输入参数。递减函数会在累加器中进行调用，并且`fn`将会对输入变量进行变换。如果这里看起来比较复杂的话，我们将在后面进行详细的解析：

   ```c++
       return [=] (auto reduce_fn) {
           return [=] (auto accum, auto input) {
           	return reduce_fn(accum, fn(input));
           };
       };
   }
   ```

4. 现在，让我们来实现一个`filter`函数。其和`map`的工作原理一样，不过其不会对输入进行修改(`map`中会对输入进行变换)。另外，我们接受一个谓词函数，并且在不接受谓词函数的情况下，跳过输入变量，而非减少输入变量：

   ```c++
   template <typename T>
   auto filter(T predicate)
   {
   ```

5. 两个Lambda表达式与`map`函数具有相同的函数签名。其不同点在于`input`参数是否进行过操作。谓词函数用来区分我们是否对输入调用`reduce_fn`函数，或者直接调用累加器而不进行任何修改：

   ```c++
       return [=] (auto reduce_fn) {
           return [=] (auto accum, auto input) {
               if (predicate(input)) {
               	return reduce_fn(accum, input);
               } else {
               	return accum;
               }
           };
       };
   }
   ```

6. 现在让我们使用这些辅助函数。我们实例化迭代器，我们会从标准输入中获取整数值：

   ```c++
   int main()
   {
       std::istream_iterator<int> it {std::cin};
       std::istream_iterator<int> end_it;
   ```

7. 然后，我们会调用谓词函数`even`，当传入一个偶数时，这个函数会返回true。变换函数`twice`会对输入整数做乘2处理：

   ```c++
       auto even ([](int i) { return i % 2 == 0; });
       auto twice ([](int i) { return i * 2; });
   ```

8. `std::accumulate`函数会将所对应范围内的数值进行累加。累加默认就是通过`+`操作符将范围内的值进行相加。我们想要提供自己的累加函数，也就是我们不想只对值进行累加。我们会将迭代器`it`进行解引用，获得其对应的值，之后对再对其进行处理：

   ```c++
       auto copy_and_advance ([](auto it, auto input) {
           *it = input;
           return ++it;
       });
   ```

9. 我们现在将之前零零散散的实现拼组在一起。我们对标准输入进行迭代，通过输出迭代器`ostream_iterator`将对应的值输出在终端上。 `copy_and_advance`函数对象将会接收用户输入的整型值，之后使用输出迭代器进行输出。将值赋值给输出迭代器，将会使打印变得高效。不过，我们只会将偶数挑出来，然后对其进行乘法操作。为了达到这个目的，我们将`copy_and_advance`函数包装入`even`过滤器中，再包装入`twice`引射器中：

   ```c++
       std::accumulate(it, end_it,
           std::ostream_iterator<int>{std::cout, ", "},
           filter(even)(
               map(twice)(
               	copy_and_advance
               )
           ));
       std::cout << '\n';
   }
   ```

10. 编译并运行程序，我们将得到如下的输出。奇数都被抛弃了，只有偶数做了乘2运算：

    ```c++
    $ echo "1 2 3 4 5 6" | ./transform_if
    4, 8, 12,
    ```

## How it works...

本节看起来还是很复杂的，因为我们使用了很多嵌套Lambda表达式。为了跟清晰的了解它们是如何工作的，我们先了解一下`std::accumulate`的内部工作原理。下面的实现类似一个标准函数的实现：

```c++
template <typename T, typename F>
T accumulate(InputIterator first, InputIterator last, T init, F f)
{
    for (; first != last; ++first) {
    	init = f(init, *first);
    }
    return init;
}
```

函数参数f在这起到主要作用，所有值都会累加到用户提供的`init`变量上。通常情况下，迭代器范围将会传入一组数字，类似`0, 1, 2, 3, 4 `，并且`init`的值为0。函数`f`只是一个二元函数，其会计算两个数的加和。

例子中循环将会将所有值累加到`init`上，也就类似于`init += (((0 + 1) + 2) + 3) + 4 `。这样看起来`std::accumulate`就是一个通用的折叠函数。折叠范围意味着，将二值操作应用于累加器变量和迭代范围内的每一个值(累加完一个数，再累加下一个数)。这个函数很通用，可以用它做很多事情，就比如实现`std::transform_if`函数！`f`函数也会递减函数中进行调用。

`transform_if`的一种很直接的实现，类似如下代码：

```c++
template <typename InputIterator, typename OutputIterator, typename P, typename Transform>
OutputIterator transform_if(InputIterator first, InputIterator last,OutputIterator out,P predicate, Transform trans)
{
    for (; first != last; ++first) {
        if (predicate(*first)) {
            *out = trans(*first);
            ++out;
        }
    }
    return out;
}
```

这个实现看起来和`std::accumulate`的实现很类似，这里的`out`参数可以看作为`init`变量，并且使用函数`f`替换`if`。

我们确实做到了。我们构建了`if`代码块，并且将二元函数对象作为一个参数提供给了`std::accumulate`：

```c++
auto copy_and_advance ([](auto it, auto input) {
    *it = input;
    return ++it;
});
```

`std::accumulate`会将`init`值作为二元函数`it`的参数传入，第二个参数则是当前迭代器所指向的数据。我们提供了一个输出迭代器作为`init`参数。这样`std::accumulate`就不会做累加，而是将其迭代的内容转发到另一个范围内。这就意味着，我们只需要重新实现`std::copy`就可以了。

通过`copy_and_advance`函数对象，使用我们提供的谓词，将过滤后的结果传入另一个使用谓词的函数对象：

```c++
template <typename T>
auto filter(T predicate)
{
    return [=] (auto reduce_fn) {
        return [=] (auto accum, auto input) {
            if (predicate(input)) {
            	return reduce_fn(accum, input);
            } else {
            	return accum;
            }
        };
    };
}
```

构建过程看上去没那么简单，不过先来看一下`if`代码块。当`predicate`函数返回true时，其将返回`reduce_fn`函数处理后的结果，也就是`accum`变量。这个实现省略了使用过滤器的操作。`if`代码块位于Lambda表达式的内部，其具有和`copy_and_advance`一样的函数签名，这使它成为一个合适的替代品。

现在我们就要进行过滤，但不进行变换。这个操作有`map`辅助函数完成：

```c++
template <typename T>
auto map(T fn)
{
    return [=] (auto reduce_fn) {
        return [=] (auto accum, auto input) {
        	return reduce_fn(accum, fn(input));
        };
    };
}
```

这段代码看起来就简单多了。其内部有一个还有一个Lambda表达式，该表达式的函数签名与`copy_and_advance`，所以可以替代`copy_and_advance`。这个实现仅转发输入变量，不过会通过二元函数对`fn`的调用，对参数进行量化。

之后，当我们使用这些辅助函数时，我们可以写成如下的表达式：

```c++
filter(even)(
    map(twice)(
    	copy_and_advance
    )
)
```

`filter(even)`将会捕获`even`谓词，并且返回给我们一个函数，其为一个包装了另一个二元函数的二元函数，被包装的那个二元函数则是进行过滤的函数。`map(twice)`函数做了相同的事情，`twice`变换函数，将`copy_and_advance`包装入另一个二元函数中，那另一个二元函数则是对参数进行变换的函数。

虽然没有任何的优化，但我们的代码还是非常的复杂。为了让函数之间能一起工作，我们对函数进行了多层嵌套。不过，这对于编译器来说不是一件很难的事情，并且能对所有代码进行优化。程序最后的结果要比实现`transform_if`简单很多。这里我们没有多花一分钱，就获得了非常好的函数模组。这里我们就像堆乐高积木一样，可将`even`谓词和`twice`转换函数相结合在一起。

