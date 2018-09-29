# 并行ASCII曼德尔布罗特渲染器——std::async

还记得第6章中的[ASCII曼德尔布罗特渲染器](content/chapter6/chapter6-5-chinese.md)吗？本节中，我们将使用多线程来加速其计算的过程。

原始代码中会限定每个坐标的迭代次数，坐标的迭代会让程序变得很慢，现在我们使用并行方式对其进行实现。

然后，我们对代码做少量的修改，并且将`std::async`和`std::future`加入到程序中，让程序运行的更快。想要完全理解本节，就要对原始的程序有个较为完整的了解。

## How to do it...

本节中，我们将对曼德尔布罗特渲染器进行升级。首先，要提升对选定坐标迭代计算的次数。然后，通过程序并行化，来提高运行的速度：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <algorithm>
   #include <iterator>
   #include <complex>
   #include <numeric>
   #include <vector>
   #include <future>
   
   using namespace std;
   ```

2. `scaler` 和`scaled_cmplx`没有任何改动：

   ```c++
   using cmplx = complex<double>;
   
   static auto scaler(int min_from, int max_from,
   					double min_to, double max_to)
   {
   	const int w_from {max_from - min_from};
   	const double w_to {max_to - min_to};
   	const int mid_from {(max_from - min_from) / 2 + min_from};
   	const double mid_to {(max_to - min_to) / 2.0 + min_to};
   
       return [=] (int from) {
   		return double(from - mid_from) / w_from * w_to + mid_to;
   	};
   }
   
   template <typename A, typename B>
   static auto scaled_cmplx(A scaler_x, B scaler_y)
   {
   	return [=](int x, int y) {
   		return cmplx{scaler_x(x), scaler_y(y)};
   	};
   }
   ```

3. `mandelbrot_iterations`函数中会增加迭代的次数，为的就是增加计算负荷：

   ```c++
   static auto mandelbrot_iterations(cmplx c)
   {
       cmplx z {};
       size_t iterations {0};
       const size_t max_iterations {100000};
       while (abs(z) < 2 && iterations < max_iterations) {
           ++iterations;
           z = pow(z, 2) + c;
       }
       return iterations;
   }
   ```

4. 主函数中的部分代码也不需要进行任何修改：

   ```c++
   int main()
   {
       const size_t w {100};
       const size_t h {40};
       
       auto scale (scaled_cmplx(
           scaler(0, w, -2.0, 1.0),
           scaler(0, h, -1.0, 1.0)
       ));
       
       auto i_to_xy ([=](int x) {
      		return scale(x % w, x / w);
       }); 
   ```

5. `to_iteration_count`函数中，不能直接调用`mandelbrot_iterations(x_to_xy(x))`，需要使用异步函数` std::async `：

   ```c++
       auto to_iteration_count ([=](int x) {
           return async(launch::async,
           			mandelbrot_iterations, i_to_xy(x));
       });	
   ```

6. 进行最后的修改之前，函数`to_iteration_count`会返回特定坐标需要迭代的次数。那么就会返回一个`future`变量，这个变量用于在后面获取异步结果时使用。因此，需要一个`vector`来盛放所有`future`变量，所以我们就在这里添加了一个。将输出迭代器作为第三个参数传入`transform`函数，并在`vector`变量`r`中放入新的输出：

   ```c++
   	vector<int> v (w * h);
       vector<future<size_t>> r (w * h);
       iota(begin(v), end(v), 0);
       transform(begin(v), end(v), begin(r),
       		 to_iteration_count);
   ```

7. `accumulate`不会在对第二个参数中`size_t`的值进行打印，不过这次改成了`future<size_t>`。我们需要花点时间对这个类型进行适应(对于一些初学者来说，这里使用`auto&`类型的话可能会让其产生疑惑)，之后需要调用`x.get()`来访问`x`中的值，如果`x`中的值还没计算出来，程序将会阻塞进行等待：

   ```c++
       auto binfunc ([w, n{0}] (auto output_it, future<size_t> &x)
       		mutable {
       	*++output_it = (x.get() > 50 ? '*' : ' ');
       	if (++n % w == 0) { ++output_it = '\n'; }
      	 	return output_it;
       });
                     
       accumulate(begin(r), end(r),
       		  ostream_iterator<char>{cout}, binfunc);
   }
   ```

8. 编译并运行程序，我们也能得到和之前一样的输出。唯一不同的就是执行的速度。我们增加了原始版本的迭代次数，程序应该会更慢，不过好在有并行化的帮助，我们能够计算的更快。我的机器上有4个CPU核，并且支持超线程(也就是有8个虚拟核)，我使用GCC和clang得到了不同结果。最好的加速效果有5.3倍，最差也有3.8倍。当然，这个结果和机器的很多状态有关。

## How it works...

理解本节代码的关键就在于下面这句和CPU强相关的代码行：

```c++
transform(begin(v), end(v), begin(r), to_iteration_count);
```

`vector v`中包含了所有复数坐标，然后这些坐标会通过曼德尔布罗特算法进行迭代。每次的迭代结果则会保存在`vector r`中。

原始代码中，我们将所要绘制的分形图形保存为一维数据。代码则会对之前所有的工作结果进行打印。这也就意味着并行化是提升性能的一个关键因素。

唯一可能并行化的部分就是从`begin(v)`到`end(v)`的处理，每块都具有相同尺寸，并能够分布在所有核上。这样所有核将会对输入数据进行共享。如果使用并行版本的`std::transform`，就需要带上一个执行策略。不幸的是，这不是问题的正确解决方式，因为每一个曼德尔布罗特集合中的点，迭代的次数是不同的。

我们的方式是使用一个`vector`收集将要获取每个点所要计算的数量的`future`变量。代码中`vector`能容纳`w * h`个元素，例子中就是`100 * 40`，也就是说`vector`实例中存储了4000个`future`变量，这些变量都会在异步计算中得到属于自己的值。如果我们的系统有4000个CPU核，就可以启动4000个并发的对坐标进行迭代计算。一个常见的机器上并没有那么多核，CPU只能是异步的对于一个元素进行处理，处理完成后再继续下一个。

`to_iteration_count`中调用异步版本的`transform`时，并不是去计算，而是对线程进行部署，然后立即获得对应的`future`对象。原始版本会在每个点上阻塞很久，因为迭代需要花费很长时间。

并行版本的程序，也有可能会在那里发生阻塞。打印函数所打印出的结果必须要从`future`对象中获取，为了完成这个目的，我们调用`x.get()`用来获取所有结果。诀窍就在这里：等待第一个值被打印时，其他值也同时在计算。所以，当调用`get()`返回时，下一个`future`的结果也会很快地被打印出来！

当`w * h`是一个非常大的数时，创建`future`对象和同步`future`对象的开销将会非常可观。本节的例子中，这里的开销并不明显。我的笔记本上有一个i7 4核超线程的CPU(也就是有8个虚拟核)，并行版本与原始版本对比有3-5倍的加速，理想的并行加速应该是8倍。当然，影响机器的因素有很多，并且不同的机器也会有不同的加速比。