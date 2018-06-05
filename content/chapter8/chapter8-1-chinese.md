# 使用std::ratio转换不同的时间单位

在C++11之后，STL具有了很多用来测量和显示时间的新类型和函数。STL的这部分内容放在std::chrono命名空间中。

本节中，我们将关注与测量时间，并且如何对两种不同的时间单位进行转换，比如秒与毫秒和微秒的转换。STL已经提供了现成的工具，其能让我们自定义时间单位，并且可以无缝的在不同的时间单位间进行转换。

## How to do it...

本节中，我们写一个小游戏，游戏会让用户输入一个单词。我们将记录用户打字的速度，并将时间以不同的时间单位显示出来：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <chrono>
   #include <ratio>
   #include <cmath>
   #include <iomanip>
   #include <optional>
   
   using namespace std; 
   ```

2.  chrono::duration经常用来表示所用时间的长度，其为秒的倍数或小数。所有STL的持续都由整型类型进行特化。本节中，我们将使用double进行特化。在本节之后，我们更多的会关注已经存在于STL的时间单位：

   ```c++
   using seconds = chrono::duration<double>;
   ```

3. 以毫秒为1/1000秒，所以我们可以用这个单位来定义秒。ratio_multiply模板参数可以使用STL预定义的milli用来表示seconds::period，其会给我们相应的小数。ratio_multiply为基本时间的倍数：

   ```c++
   using milliseconds = chrono::duration<
   	double, ratio_multiply<seconds::period, milli>>;
   ```

4. 对于微秒来说也是一样的。我们可以使用micro表示：

   ```c++
   using microseconds = chrono::duration<
   	double, ratio_multiply<seconds::period, micro>>;
   ```

5. 现在我们会实现一个函数，其会用来从用户的输入中读取一个字符串，并且测量用户输入所用的时间。这个函数并没有参数，并且返回用户输入的同时，返回所用的时间，我们用一个组对将这两个数进行返回：

   ```c++
   static pair<string, seconds> get_input()
   {
   	string s;
   ```

6. 我们需要从用户开始输入时计时，记录一个时间点的方式可以写成如下方式：

   ```c++
   	const auto tic (chrono::steady_clock::now());
   ```

7. 现在可以来获取用户的输入了。当我们没有获取成功，我们将会返回一个默认的元组对象。这个元组对象中的元素都是空的：

   ```c++
       if (!(cin >> s)) {
       	return {{}, {}};
       }
   ```

8. 当成功的获取输入后，我们会打上下一个时间点。然后，返回用户的输入和输入所用的时间。注意这里获取的都是绝对的时间点，通过计算这两个时间点的差，我们得到了打印所用的时间：

   ```c++
       const auto toc (chrono::steady_clock::now());
       
   	return {s, toc - tic};
   } 
   ```

9. 现在让我们来实现主函数。我们使用一个循环获取用户的输入，这个循环直到用户输入正确的字符串为止。在每次循环中，我们都会让用户输入"C++17"，然后调用我们的get_input函数：

   ```c++
   int main()
   {
       while (true) {
       	cout << "Please type the word \"C++17\" as"
       			" fast as you can.\n> ";
           
       	const auto [user_input, diff] = get_input();
   ```

10. 然后我们对输入进行检查。当输入为空，我们会让程序终止：

    ```c++
    		if (user_input == "") { break; }
    ```

11. 当用户正确的输入"C++17"，我们将会对用户表示祝贺，然后返回其输入所用时间。diff.count()函数会以浮点数的方式返回输入所用的时间。当我们使用STL原始的seconds时间类型是，我们将会得到一个已舍入的整数，而不是一个小数。通过使用毫秒和微秒我们将获得对应单位的计数，然后我们通过相应的转换方式进行时间单位变换：

    ```c++
            if (user_input == "C++17") {
                cout << "Bravo. You did it in:\n"
                    << fixed << setprecision(2)
                    << setw(12) << diff.count()
                    << " seconds.\n"
                    << setw(12) << milliseconds(diff).count()
                    << " milliseconds.\n"
                    << setw(12) << microseconds(diff).count()
                    << " microseconds.\n";
                break;
    ```

12. 如果用户输入有误时，我们会提示其继续输入：

    ```c++
            } else {
                cout << "Sorry, your input does not match."
               			" You may try again.\n";
            }
        }
    }
    ```

13. 编译并运行程序，就会得到如下的输出。第一次输入时，会有一个错误，程序会让我们重新进行输入。在正确输入之后，我们就会得到我们输入所花费的时间：

    ```c++
    $ ./ratio_conversion
    Please type the word "C++17" as fast as you can.
    > c+17
    Sorry, your input does not match. You may try again.
    Please type the word "C++17" as fast as you can.
    > C++17
    Bravo. You did it in:
    1.48 seconds.
    1480.10 milliseconds.
    1480099.00 microseconds.
    ```

## How it works...

本节中对不同时间单位进行转换是，我们需要先选择三个可用的时钟对象的一个。其分别为system_clock，steady_clock和high_resolution_clock，这三个时钟对象都在std::chrono命名空间中。他们有什么区别呢？让我们来看一下：

| 时钟类型              | 特性                                                         |
| --------------------- | ------------------------------------------------------------ |
| system_clock          | 其表示系统级别实时的挂钟。当我们想要获取本地时间，这是个正确的选择。 |
| steady_clock          | 该时钟表示单调型的时间。这就表示这个时间是不可能倒退的。而时间倒退可能会在其他时钟上发生，当其最小精度不同，或是在冬令时和夏令时交替时。 |
| high_resolution_clock | STL中可统计最细粒度时钟周期的时钟。                          |

当我们要衡量时间的距离，或者计算两个时间点的绝对间隔。即便时钟是112年，5小时，10分钟，1秒(或其他)之后，或之前的时间，这都不影响两个时间点间的相对距离。这里我们唯一关注的就是打的两个时间点toc和tic，这里的时钟需要是微秒级别的(许多系统都使用这样的时钟)，因为不同的时钟对于我们的测量有一定的影响。对于这样的需求，steady_clock无疑是最佳的选择。其能根据处理器的时间戳计数器进行实现，只要该时钟开始计数(系统开始运行)就不会停止。

OK，现在来对合适的时间对象进行选择，我们可以通过`chrono::steady_clock::now()`对时间点进行保存。now函数会返回一个`chrono::time_point<chrono::steady_clock>`类的值。

## There's more...

