# 快速或安全的访问std::vector实例的方法

`std::vector`可能是STL容器中适用范围最广的，因为其存储数据的方式和数组一样，并且还有相对完善的配套设施。不过，非法访问一个`vector`实例还是十分危险的。如果一个`vector`实例具有100个元素，那当我们想要访问索引为123的元素时，程序就会崩溃掉。如果不崩溃，那么你就麻烦了，未定义的行为会导致一系列奇奇怪怪的错误，查都不好查。经验丰富的开发者会在访问前，对索引进行检查。这样的检查其实比较多余，因为很多人不知道`std::vector`有内置的检查机制。

## How to do it...

本节我们将使用两种不同的方式访问一个`std::vector`实例，并且利用其特性编写更加安全的代码。

1. 先包含相应的头文件，并且用1000个123填满一个vector实例：

   ```c++
   #include <iostream>
   #include <vector>
   using namespace std;
   int main()
   {
       const size_t container_size{1000};
       vector<int> v(container_size, 123);
   ```

2. 我们通过`[]`操作符访问范围之外的元素：

   ```c++
       cout << "Out of range element value: "
            << v[container_size + 10] << '\n';
   ```

3. 之后我们使用`at`函数访问范围之外的元素：

   ```c++
       cout << "Out of range element value: "
            << v.at(container_size + 10) << '\n';
   }
   ```

4. 让我们运行程序，看下会发生什么。下面的错误信息是由GCC给出。其他编译器也会通过不同方式给出类似的错误提示。第一种方式得到的结果比较奇怪。超出范围的访问方式并没有让程序崩溃，但是访问到了与123相差很大的数字。第二种方式中，我们看不到打印出来的结果，因为在打印之前程序已经崩溃了。当越界访问发生的时候，我们可以通过异常的方式更早的得知！

   ```c++
   Out of range element value: -726629391
   terminate called after throwing an instance of 'std::out_of_range'
   what(): array::at: __n (which is 1010) >= _Nm (which is 1000)
   Aborted (core dumped)
   ```

## How it works...

`std::vector`提供了`[]`操作符和`at`函数，它们的作用几乎是一样的。`at`函数会检查给定的索引值是否越界，如果越界则返回一个异常。这对于很多情景都十分适用，不过因为检查越界要花费一些时间，所以`at`函数会让程序慢一些。

当需要非常快的索引成员时，并能保证索引不越界，我们会使用`[]`快速访问`vector`实例。很多情况下，`at`函数在牺牲一点性能的基础上，有助于发现程序内在的bug。

> Note：
>
> 默认使用`at`函数是一个好习惯。当代码的性能很差，但没有bug存在时，可以使用性能更高的操作符来替代`at`函数。

## There's more...

当然，我们需要处理越界访问，避免整个程序崩溃。为了对越界访问进行处理，我们可以使用截获异常的方式。可以用`try`代码块将调用at函数的部分包围，并且定义错误处理的`catch`代码段。

```c++
try {
	std::cout << "Out of range element value: "
        	  << v.at(container_size + 10) << '\n';
} catch (const std::out_of_range &e) {
	std::cout << "Ooops, out of range access detected: "
              << e.what() << '\n';
}
```

> Note:
>
> 顺带一提，`std::array`也提供了`at`函数。

