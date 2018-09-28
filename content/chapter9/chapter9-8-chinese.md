# 将执行的程序推到后台——std::async

当我们想要将一些可以执行的代码放在后台，可以用线程将这段程序运行起来。然后，我们就等待运行的结果就好：

```c++
std::thread t {my_function, arg1, arg2, ...};
// do something else
t.join(); // wait for thread to finish
```

这里`t.join()`并不会给我们`my_function`函数的返回值。为了获取返回值，需要先实现`my_function`函数，然后将其返回值存储到主线程能访问到的地方。如果这样的情况经常发生，我们就要重复的写很多代码。

C++11之后，`std::async`能帮我们完成这项任务。我们将写一个简单的程序，并使用异步函数，让线程在同一时间内做很多事情。`std::async`其实很强大，让我们先来了解其一方面。

## How to do it...

我们将在一个程序中并发进行多个不同事情，不显式创建线程，这次使用`std::async`和`std::future`：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <iomanip>
   #include <map>
   #include <string>
   #include <algorithm>
   #include <iterator>
   #include <future>
   
   using namespace std;
   ```

2. 实现了三个函数，算是完成些很有趣的任务。第一个函数能够接收一个字符串，并且创建一个对于字符串中的字符进行统计的直方图：

   ```c++
   static map<char, size_t> histogram(const string &s)
   {
       map<char, size_t> m;
       
       for (char c : s) { m[c] += 1; }
       
       return m;
   }
   ```

3. 第二个函数也能接收一个字符串，并返回一个排序后的副本：

   ```c++
   static string sorted(string s)
   {
       sort(begin(s), end(s));
       return s;
   }
   ```

4. 第三个函数会对传入的字符串中元音字母进行计数：

   ```c++
   static bool is_vowel(char c)
   {
       char vowels[] {"aeiou"};
       return end(vowels) !=
       		find(begin(vowels), end(vowels), c);
   }
   
   static size_t vowels(const string &s)
   {
   	return count_if(begin(s), end(s), is_vowel);
   }
   ```

5. 主函数中，我们从标准输入中获取字符串。为了不让输入字符串分段，我们禁用了`ios::skipws`。这样就能得到一个很长的字符串，并且不管这个字符串中有多少个空格。我们会对结果字符串使用`pop_back`，因为这种方式会让一个字符串中包含太多的终止符：

   ```c++
   int main()
   {
       cin.unsetf(ios::skipws);
       string input {istream_iterator<char>{cin}, {}};
       input.pop_back();
   ```

6. 为了获取函数的返回值，并加快对输入字符串的处理速度，我们使用了异步的方式。`std::async`函数能够接收一个策略和一个函数，以及函数对应的参数。我们对于这个三个函数均使用`launch::async`策略。并且，三个函数的输入参数是完全相同的：

   ```c++
   	auto hist (async(launch::async,
       				histogram, input));
       auto sorted_str (async(launch::async,
       				sorted, input));
       auto vowel_count (async(launch::async,
       				vowels, input));
   ```

7. `async`的调用会立即返回，因为其并没有执行我们的函数。另外，准备好同步的结构体，用来获取函数所返回的结果。目前的结果使用不同的线程并发的进行计算。此时，我们可以做其他事情，之后再来获取函数的返回值。`hist`，`sorted_str`和`vowel_count`分别为函数`histogram`，`sorted` 和`vowels`的返回值，不过其会通过`std::async`包装入`future`类型中。这个对象表示在未来某个时间点上，对象将会获取返回值。通过对`future`对象使用`.get()`，我们将会阻塞主函数，直到相应的值返回，然后再进行打印：

   ```c++
       for (const auto &[c, count] : hist.get()) {
       	cout << c << ": " << count << '\n';
       }
   
       cout << "Sorted string: "
           << quoted(sorted_str.get()) << '\n'
           << "Total vowels: "
           << vowel_count.get() << '\n';
   }
   ```

8. 编译并运行代码，就能得到如下的输出。我们使用一个简短的字符串的例子时，代码并不是真正的在并行，但这个例子中，我们能确保代码是并发的。另外，程序的结构与串行版本相比，并没有改变多少：

    ```c++
    $ echo "foo bar baz foobazinga" | ./async
     : 3
    a: 4
    b: 3
    f: 2
    g: 1
    i: 1
    n: 1
    o: 4
    r: 1
    z: 2
    Sorted string: "   aaaabbbffginoooorzz"
    Total vowels: 9
    ```

## How it works...

如果你没有使用过`std::async`，那么代码可以简单的写成串行代码：

```c++
auto hist (histogram(input));
auto sorted_str (sorted( input));
auto vowel_count (vowels( input));

for (const auto &[c, count] : hist) {
  cout << c << ": " << count << '\n';
}
cout << "Sorted string: " << quoted(sorted_str) << '\n';
cout << "Total vowels: " << vowel_count << '\n';
```

下面的代码，则是并行的版本。我们将三个函数使用`async(launch::async, ...)`进行包装。这样三个函数都不会由主函数来完成。此外，`async`会启动新线程，并让线程并发的完成这几个函数。这样我们只需要启动一个线程的开销，就能将对应的工作放在后台进行，而后可以继续执行其他代码：

```c++
auto hist (async(launch::async, histogram, input));
auto sorted_str (async(launch::async, sorted, input));
auto vowel_count (async(launch::async, vowels, input));

for (const auto &[c, count] : hist.get()) {
	cout << c << ": " << count << '\n';
}

cout << "Sorted string: "
    << quoted(sorted_str.get()) << '\n'
    << "Total vowels: "
    << vowel_count.get() << '\n';
```

例如`histogram`函数则会返回一个`map`实例，`async(..., histogram, ...)`将返回给我们的`map`实例包装进之前就准备好的`future`对象中。`future`对象时一种空的占位符，直到线程执行完函数返回时，才有具体的值。结果`map`将会返回到`future`对象中，所以我们可以对对象进行访问。`get`函数能让我们得到被包装起来的结果。

让我们来看一个更加简单的例子。看一下下面的代码：

```c++
auto x (f(1, 2, 3));
cout << x;
```

与之前的代码相比，我们也可以以下面的方式完成代码：

```c++
auto x (async(launch::async, f, 1, 2, 3));
cout << x.get();
```

这都是最基本的。后台执行的方式可能要比标准C++出现还要早。当然，还有一个问题要解决：`launch::async`是什么东西？` launch::async`是一个用来定义执行策略的标识。其有两种独立方式和一种组合方式：

| 策略选择                                                     | 意义                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [launch::async](http://zh.cppreference.com/w/cpp/thread/launch) | 运行新线程，以异步执行任务                                   |
| [launch::deferred](http://zh.cppreference.com/w/cpp/thread/launch) | 在调用线程上执行任务(惰性求值)。在对`future`调用`get`和`wait`的时候，才进行执行。如果什么都没有发生，那么执行函数就没有运行。 |
| launch::async \| launch::deferred                            | 具有两种策略共同的特性，STL的`async`实现可以的选择策略。当没有提供策略时，这种策略就作为默认的选择。 |

> Note：
>
> 不使用策略参数调用`async(f, 1, 2, 3)`，我们将会选择都是用的策略。`async`的实现可以自由的选择策略。这也就意味着，我们不能确定任务会执行在一个新的线程上，还是执行在当前线程上。

## There's more...

还有件事情我们必须要知道，假设我们写了如下的代码：

```c++
async(launch::async, f);
async(launch::async, g);
```

这就会让`f`和`g`函数并发执行(这个例子中，我们并不关心其返回值)。运行这段代码时，代码会阻塞在这两个调用上，这并不是我们想看到的情况。

所以，为什么会阻塞呢？`async`不是非阻塞式、异步的调用吗？没错，不过这里有点特殊：当对一个`async`使用`launch::async`策略时，获取一个`future`对象，之后其析构函数将会以阻塞式等待方式运行。

这也就意味着，这两次调用阻塞的原因就是，`future`生命周期只有一行的时间！我们可以以获取其返回值的方式，来避免这个问题，从而让`future`对象的生命周期更长。