# 迭代器进行打印——std::ostream

使用输出流进行打印是一件很容易的事情，STL中的大多数基本类型都对`operator<<`操作符进行过重载。所以使用`std::ostream_iterator`类，就可以将数据类型中所具有的的元素进行打印，我们已经在之前的章节中这样做了。

本节中，我们将关注如何将自定义的类型进行打印，并且可以通过模板类进行控制。对于调用者来说，无需写太多的代码。

## How to do it...

我们将对一个新的自定义的类使用`std::ostream_iterator`，并且看起来其具有隐式转换的能力，这就能帮助我们进行打印：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <vector>
   #include <iterator>
   #include <unordered_map>
   #include <algorithm>
   
   using namespace std;
   using namespace std::string_literals;
   ```

2. 让我们实现一个转换函数，其会将数字和字符串相对应。比如输入1，就会返回“one”；输入2，就会返回“two”，以此类推：

   ```c++
   string word_num(int i) {
   ```

3. 将会对哈希表进行填充，我们后续可以对它进行访问：

   ```c++
   	unordered_map<int, string> m {
           {1, "one"}, {2, "two"}, {3, "three"},
           {4, "four"}, {5, "five"}, //...
   	};
   ```

4. 现在可以使用哈希表的`find`成员函数，通过传入相应的键值，返回对应的值。如果`find`函数找不到任何东西，我们就会得到一个"unknown"字符串：

   ```c++
       const auto match (m.find(i));
       if (match == end(m)) { return "unknown"; }
       return match->second;
   }; 
   ```

5. 接下来我们就要定义一个结构体`bork`。其仅包含一个整型成员，其可以使用一个整型变量进行隐式构造。其具有`print`函数，其能接受一个输出流引用，通过`borks`结构体的整型成员变量，重复打印"bork"字符串：

   ```c++
   struct bork {
       int borks;
       
       bork(int i) : borks{i} {}
       
       void print(ostream& os) const {
           fill_n(ostream_iterator<string>{os, " "},
           	   borks, "bork!"s);
       }
   };
   ```

6. 为了能够更方便的对`bork`进行打印，对`operator<<`进行了重载，当通过输出流对`bork`进行输出时，其会自动的调用`bork::print`：

   ```c++
   ostream& operator<<(ostream &os, const bork &b) {
       b.print(os);
       return os;
   }
   ```

7. 现在来实现主函数，先来初始化一个`vector`:

   ```c++
   int main()
   {
   	const vector<int> v {1, 2, 3, 4, 5};
   ```

8. `ostream_iterator`需要一个模板参数，其能够表述哪种类型的变量我们能够进行打印。当使用`ostream_iterator<T>`时，其会使用` ostream& operator(ostream&, const T&)`进行打印。这也就是之前在`bork`类型中重载的输出流操作符。我们这次只对整型数字进行打印，所以使用`ostream_iterator<int> `。使用`cout`进行打印，并可以将其作为构造参数。我们使用循环对`vector`进行访问，并且对每个输出迭代器`i`进行解引用。这也就是在STL算法中流迭代器的用法：

   ```c++
   	ostream_iterator<int> oit {cout};
       for (int i : v) { *oit = i; }
       cout << '\n';
   ```

9. 使用的输出迭代器还不错，不过其打印没有任何分隔符。当需要空格分隔符对所有打印的元素进行分隔时，我们可以将空格作为第二个参数传入输出流构造函数中。这样，其就能打印"1, 2, 3, 4, 5, "，而非"12345"。不过，不能在打印最后一个数字的时候将“逗号-空格”的字符串丢弃，因为迭代器并不知道哪个数字是最后一个：

   ```c++
       ostream_iterator<int> oit_comma {cout, ", "};
       
   	for (int i : v) { *oit_comma = i; }
       cout << '\n';
   ```

10. 为了将其进行打印，我们将值赋予一个输出流迭代器。这个方法可以和算法进行结合，其中最简单的方式就是`std::copy`。我们可以通过提供`begin`和`end`迭代器来代表输入的范围，在提供输出流迭代器作为输出迭代器。其将打印`vector`中的所有值。这里我们会将两个输出循环进行比较：

   ```c++
   	copy(begin(v), end(v), oit);
   	cout << '\n';
   
   	copy(begin(v), end(v), oit_comma);
   	cout << '\n';
   ```

11. 还记得`word_num`函数吗？其会将数字和字符串进行对应。我们也可以使用进行打印。我们只需要使用一个输出流操作符，因为我们不需要对整型变量进行打印，所以这里使用的是`string`的特化版本。使用`std::transfrom`替代`std::copy`，因为需要使用转换函数将输入范围内的值转换成其他值，然后拷贝到输出中：

    ```c++
        transform(begin(v), end(v),
        		 ostream_iterator<string>{cout, " "}, word_num);
        cout << '\n';
    ```

12. 程序的最后一行会对`bork`结构体进行打印。可以直接使用，也并不需要为`std::transform`函数提供任何转换函数。另外，可以创建一个输出流迭代器，其会使用`bork`进行特化，然后再调用`std::copy`。`bork`实例可以通过输入范围内的整型数字进行隐式创建。然后，将会得到一些有趣的输出：

    ```c++
        copy(begin(v), end(v),
        	 ostream_iterator<bork>{cout, "\n"});
    }
    ```

13. 编译并运行程序，就会得到以下输出。前两行和第三四行的结果非常类似。然后，会得到数字对应的字符串，然后就会得到一堆`bork!`字符串。其会打印很多行，因为我们使用换行符替换了空格：

    ```c++
    $ ./ostream_printing
    12345
    1, 2, 3, 4, 5,
    12345
    1, 2, 3, 4, 5,
    one two three four five
    bork!
    bork! bork!
    bork! bork! bork!
    bork! bork! bork! bork!
    bork! bork! bork! bork! bork!
    ```

## How it works...

作为一个语法黑客，我们应知道`std::ostream_iterator`可以用来对数据进行打印，其在语法上为一个迭代器，对这个迭代器进行累加是无效的。对其进行解引用会返回一个代理对象，这些赋值操作符会将这些数字转发到输出流中。

输出流迭代器会对类型T进行特化(`ostream_iterator<T> `)，对于所有类型的` ostream& operator<<(ostream&, const T&)`来说，都需要对其进行实现。

`ostream_iterator`总是会调用` operator<<`，通过模板参数，我们已经对相应类型进行了特化。如果类型允许，这其中会发生隐式转换。当A可以隐式转换为B时，我们可以对A类型的元素进行迭代，然后将这些元素拷贝到` output_iterator<B>`的实例中。我们会对`bork`结构体做同样的事情：`bork`实例也可以隐式转换为一个整数，这也就是我们能够很容易的在终端输出一堆`bork!`的原因。

如果不能进行隐式转换，可使用`std::treansform`和`word_num`函数相结合，对元素类型进行转换。

> Note：
>
> 通常，对于自定义类型来说，隐式转换是一种不好的习惯，因为这是一个常见的Bug源，并且这种Bug非常难找。例子中，隐式构造函数有用程度要超过其危险程度，因为相应的类只是进行打印。