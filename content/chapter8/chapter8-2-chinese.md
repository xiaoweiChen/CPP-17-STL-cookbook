# 转换绝对时间和相对时间——std::chrono

C++11之前，想要获取时间并对其打印是有些困难的，因为C++并没有标准时间库。想要对时间进行统计就需要调用C库，并且我们要考虑这样的调用是否能很好的封装到我们的类中。

C++11之后，STL提供了`chrono`库，其让对时间的操作更加简单。

本节，我们将会使用本地时间，并对本地时间进行打印，还会给时间加上不同的偏移，这些操作很容易使用`std::chrono`完成。

## How to do it...

我们将会对当前时间进行保存，并对其进行打印。另外，我们的程序还会为已经保存的时间点添加不同的偏移，并且打印偏移之后的时间：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <iomanip>
   #include <chrono>
   
   using namespace std; 
   ```

2. 我们将打印绝对时间点。使用`chrono::time_point`模板类型来获取，因此需要对输出流操作符进行重载。对时间点的打印方式有很多，我们将会使用`%c`来表示标准时间格式。当然，可以只打印时间、日期或是我们需要的信息。调用`put_time`之前对不同类型的变量进行转换的方式看起来有些笨拙，不过这里只这么做一次：

   ```c++
   ostream& operator<<(ostream &os,
   	const chrono::time_point<chrono::system_clock> &t)
   {
       const auto tt (chrono::system_clock::to_time_t(t));
       const auto loct (std::localtime(&tt));
       return os << put_time(loct, "%c");
   }
   ```

3. STL已经定义了`seconds`，`minutes`，`hours`等时间类型，所以我们只需要为其添加`days`类型就好。这很简单，只需要对`chrono::duration`模板类型进行特化，将hours类型乘以24，就表示一天具有24个小时：

   ```c++
   using days = chrono::duration<
       chrono::hours::rep,
       ratio_multiply<chrono::hours::period, ratio<24>>>;
   ```

4. 为了用很有优雅的方式表示很多天，我们定义属于`days`类型的字面值操作符。现在我们程序中写`3_days`就代表着3天：

   ```c++
   constexpr days operator ""_days(unsigned long long d)
   {
   	return days{d};
   }
   ```

5. 实际程序中，我们将会对时间点进行记录，然后就会对这个时间点进行打印。因为已经对操作符进行了重载，所以完成这样的事情就变得很简单：

   ```c++
   int main()
   {
       auto now (chrono::system_clock::now());
       
       cout << "The current date and time is " << now << '\n'; 
   ```

6. 我们使用`now`函数来获得当前的时间点，并可以为这个时间添加一个偏移，然后再对其进行打印。为当前的时间添加12个小时，其表示的为12个小时之后的时间：

   ```c++
   	chrono::hours chrono_12h {12};
   	
   	cout << "In 12 hours, it will be "
   		<< (now + chrono_12h)<< '\n';
   ```

7. 这里将使用`chrono_literals`命名空间中的函数，声明使用这个命名空间会解锁小时、秒等等时间类型的间隔字面值类型。这样我们就能很优雅的对12个小时15分之前的时间或7天之前的时间进行打印：

   ```c++
       using namespace chrono_literals;
       
   	cout << "12 hours and 15 minutes ago, it was "
           << (now - 12h - 15min) << '\n'
           << "1 week ago, it was "
           << (now - 7_days) << '\n';
   } 
   ```

8. 编译并运行程序，我们将会获得如下的输出。因为使用`%c`格式对时间进行打印，所以得到还不错的时间输出格式。通过对不同格式的的字符串进行操作，我们可以获得想要的格式。要注意的是，这里的时间格式并不是以12小时AM/PM方式表示，因为程序运行在欧洲操作系统上，所以使用24小时表示的方式：

   ```c++
   $ ./relative_absolute_times
   The current date and time is Fri May 5 13:20:38 2017
   In 12 hours, it will be Sat May6 01:20:38 2017
   12 hours and 15 minutes ago, it was Fri May5 01:05:38 2017
   1 week ago, it was Fri Apr 28 13:20:38 2017
   ```

## How it works...

我们可以通过`std::chrono::system_clock`来获取当前时间点。这个STL时钟类是唯一一个能将时间点的值转换成一个时间结构体的类型，其能将时间点以能够看懂的方式进行输出。

为了打印这样的时间点，我们可以对`operator<<`操作符进行重载：

```c++
ostream& operator<<(ostream &os,
				   const chrono::time_point<chrono::system_clock> &t)
{
    const auto tt (chrono::system_clock::to_time_t(t));
    const auto loct (std::localtime(&tt));
    return os << put_time(loct, "%c");
}
```

首先，将`chrono::time_point<chrono::system_clock>`转换为`std::time_t`。然后，使用`std::localtime`将这个时间值进行转换，这样就能获取到一个本地时钟的相对时间值。这个函数会给我们返回一个转换后的指针(对于这个指针背后的内存不用我们多操心，因为其是一个静态对象，并不是从堆上分配的内存)，这样我们就能完成最终的打印。

`std::put_time`函数接受一个流对象和一个时间格式字符串。`%c`表示标准时间格式字符串，例如`Sun Mar 12 11:33:40 2017 `。我们也可以写成`%m/%d/%y`；之后，时间就会按照这个格式进行打印，比如`03/12/17`。时间格式符的描述很长，想要了解其具体描述的最好方式就是去查看C++参考手册。

除了打印，我们也会为我们的时间点添加偏移。这也很简单，比如：12小时15分钟就可以表示为`12h+15min`。`chrono_literals`命名空间为我们提供了字面类型：`hours(h), minutes(min), seconds(s), milliseconds(ms), microseconds(us), nanoseconds(ns)`。

通过对两个时间间隔的相加，我们会得到一个新的时间点。要实现这样的操作就需要对左加`operator+`和左减`operator-`操作符进行重载，这样对时间偏移的操作就会变得非常简单。