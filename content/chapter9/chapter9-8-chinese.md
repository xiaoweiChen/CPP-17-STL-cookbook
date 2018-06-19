# 将执行的程序推到后台——std::async

当我们想要将一些可以执行的代码放在后台，我们可以用线程将这段程序运行起来。然后，我们就等待运行的结果就好：

```c++
std::thread t {my_function, arg1, arg2, ...};
// do something else
t.join(); // wait for thread to finish
```

这里`t.join()`bin该不会给我们`my_function`函数的返回值。为了获取返回值，我们需要先实现`my_function`函数，然后将其返回值存储到主线程能访问到的地方。如果这样的情况经常发生，我们就要重复的写很多代码。

C++11之后，我们有了`std::async`，就能帮我们完成这项任务。我们将会写一个简单的程序，并使用一步函数，让线程在同一时间内做很多事情。`std::async`其实很强大，让我们先来了解其的一方面。

## How to do it...

这次我们将在一个程序中并发完成多个不同事情，不显式创建线程，这次我们使用`std::async`和`std::future`：

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

2. 我们实现了三个函数，其在并行中并没有做什么事，但是其完成了很有趣的任务。第一个函数能够接收一个字符串，并且创建一个对于字符串中的字符进行统计的直方图：

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

5. 主函数中，我们从标准输入中获取字符串。为了不让输入字符串分段，我们禁用了`ios::skipws`。这样我们就能得到一个很长的字符串，并且不管这个字符串中有多少个空格。我们会对结果字符串使用`pop_back`，因为这种方式会让一个字符串中包含太多的终止符：

   ```c++
   int main()
   {
       cin.unsetf(ios::skipws);
       string input {istream_iterator<char>{cin}, {}};
       input.pop_back();
   ```

6. 那么现在，就让我们获取我们实现函数的返回值。为了加快对输入字符串的处理速度，我们使用了异步的方式。`std::async`函数能够接受一个策略，一个函数，以及函数对应的参数。我们对于这个三个函数均使用`launch::async`策略。并且，三个函数的输入参数是完全相同的：

   ```c++
   	auto hist (async(launch::async,
       				histogram, input));
       auto sorted_str (async(launch::async,
       				sorted, input));
       auto vowel_count (async(launch::async,
       				vowels, input));
   ```

7. 对`async`的调用会立即返回，因为其并没有执行我们的函数。另外，其会准备好同步的结构体，用来获取函数所返回的结果。目前的结果使用不同的线程并发的进行计算。与此同时，我们可以做其他事情，在后面我们再来获取函数的返回值。`hist`，`sorted_str`和`vowel_count`分别为函数`histogram`，`sorted` 和`vowels`的返回值，不过其会通过`std::async`包装入`future`类型中。这个对象的则表示，在未来某个时间点上，对象将会获取返回值。通过对`future`对象使用`.get()`，我们将会阻塞主函数，直到相应的值返回，然后进行打印：

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
   Sorted string: " aaaabbbffginoooorzz"
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

下面的代码，则是我们并行的版本。我们将三个函数使用`async(launch::async, ...)`进行包装。这样三个函数都不会由主函数来完成。此外，`async`会启动新线程，并让线程并发的完成这几个函数。这样我们只有启动一个线程的开销，将对应的工作放在后台进行，而后可以继续执行其他代码：

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





## There's more...

