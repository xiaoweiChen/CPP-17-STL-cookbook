# 让程序在特定时间休眠

C++11中对于线程的控制非常优雅和简单。在`this_thread`的命名空间中，包含了只能被运行线程调用的函数。其包含了两个不同的函数，让线程睡眠一段时间，这样就不需要使用任何额外的库，或是操作系统依赖的库来执行这个任务。

本节中，我们将关注于如何将线程暂停一段时间，或是让其休眠一段时间。

## How to do it...

我们将完成一个短小的程序，并让主线程休眠一段时间：

1. 包含必要的头文件，并声明所使用的命名空间。`chrono_literals`空间包含一段时间的缩略值：

   ```c++
   #include <iostream>
   #include <chrono>
   #include <thread>
   
   using namespace std;
   using namespace chrono_literals; 
   ```

2. 我们直接写主函数，并让主线程休眠5秒和300毫秒。感谢`chrono_literals`，我们可以写成一种非常可读方式：

   ```c++
   int main()
   {
       cout << "Going to sleep for 5 seconds"
       	    " and 300 milli seconds.\n";
       
       this_thread::sleep_for(5s + 300ms);
   ```

3. 休眠状态是相对的。当然，我们能用绝对时间来表示。让休眠直到某个时间点才终止，这里我们在`now`的基础上加了3秒：

   ```c++
   	cout << "Going to sleep for another 3 seconds.\n";
   
   	this_thread::sleep_until(
   		chrono::high_resolution_clock::now() + 3s);
   ```

4. 在程序退出之前，我们会打印一个表示睡眠结束：

   ```c++
   	cout << "That's it.\n";
   }
   ```

5. 编译并运行程序，我们就能获得如下的输出。Linux，Mac或其他类似UNIX的操作系统会提供time命令，其能对一个可运行程序的耗时进行统计。使用time对我们的程序进行耗时统计，其告诉我们花费了8.32秒，因为我们让主线程休眠了5.3秒和3秒。最后还有一个打印，用来告诉我们主函数的休眠终止：

   ```c++
   $ time ./sleep
   Going to sleep for 5 seconds and 300 milli seconds.
   Going to sleep for another 3 seconds.
   That's it.
       
   real0m8.320s
   user0m0.005s
   sys 0m0.003s
   ```

## How it works...

`sleep_for`和`sleep_until`函数都已经在C++11中加入，存放于`std::this_thread`命名空间中。其能对当前线程进行限时阻塞(并不是整个程序或整个进程)。线程被阻塞时不会消耗CPU时间，操作系统会将其标记挂起的状态，时间到了后线程会自动醒来。这种方式的好处在于，不需要知道操作系统对我们运行的程序做了什么，因为STL会将其中的细节进行包装。

`this_thread::sleep_for`函数能够接受一个`chrono::duration`值。最简单的方式就是`1s`或`5s+300ms`。为了使用这种非常简洁的字面时间表示方式，我们需要对命名空间进行声明`using namespace std::chrono_literals;`。

`this_thread::sleep_until`函数能够接受一个`chrono::time_out`参数。这就能够简单的指定对应的壁挂钟时间，来限定线程休眠的时间。

唤醒时间和操作系统的时间的精度一样。大多数操作系统的时间精度通常够用，但是其可能对于一些时间敏感的应用非常不利。

另一种让线程休眠一段时间的方式是使用`this_thread::yield`。其没有参数，也就意味着这个函数不知道这个线程要休眠多长时间。所以，这个函数并不建议用来对线程进行休眠或停滞一个线程。其只会以协作的方式告诉操作系统，让操作系统对线程和进程重新进行调度。如果没有其他可以调度的线程或进程，那么这个“休眠”线程则会立即执行。正因为如此，很少用`yield`让线程休眠一段时间。