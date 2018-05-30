# 计算文件中的单词数量

我们在读取一个文件的时候，也想知道这个文件中包含的单词数量。我们的定义的单词时位于两个空格之间的字符组合。如何进行统计呢？

因为我们对单词的定义， 我们可以统计空格的数量。例如句子"John has a funny little dog."，这里机有五个空格，所以我们可以说这句话有六个单词。

但如果我们的句子中有空格干扰怎么办，例如："   John   has    \t   a\nfunny little dog  ."。这句中有很多不必要的空格、制表符和换行符。本书的其他章节中，我们已经了解如何将多余空格从字符串中去掉。所以，我们可以对字符串进行预处理，将不必要的空格都去掉。这样做的确可行，不过我们有一个更加简单的方法。我们为什么不用STL来帮助我们呢？

为了寻找最优的解决方案，我们将让用户选择，是从标准输入中获取数据，还是从文本文件中获取数据。

## How to do it...

本节我们将完成一个单行统计函数，其可以对输入的数据进行计数，数据源的具体方式我们可以让用户来选择。

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <fstream>
   #include <string>
   #include <algorithm>
   #include <iterator>

   using namespace std;
   ```

2. wordcount函数能接受一个输入流，例如cin。其能创建一个std::input_iterator迭代器，其能对输出字符进行标记，然后交由std::distance进行计算。distance接受两个迭代器作为参数，并确定从一个迭代器到另一个迭代器要用多少步。对于随机访问迭代器，因为有减法操作符的存在，所以实现起来非常简单。其迭代器如同指针一样，可以直接进行减法，计算出两点的距离。不过istream_iterator就不行，因为其是前向迭代器，只能向前读取，直至结束。最后所需要的步数也就是单词的数量：

   ```c++
   template <typename T>
   size_t wordcount(T &is)
   {
   	return distance(istream_iterator<string>{is}, {});
   }
   ```

3. 我们的主函数中，我们会让用户来选择输入源：

   ```c++
   int main(int argc, char **argv)
   {
   	size_t wc;
   ```

4. 如果用户选择使用文件进行输入(例如：`./count_all_words some_textfile.txt`)，我们可以通过argv获取命令行中的文件名称，并将文件打开，读取数据，从而对其文本进行单词统计：

   ```c++
   	if (argc == 2) {
   		ifstream ifs {argv[1]};
   		wc = wordcount(ifs);
   ```

5. 如果用户没有传入任何参数，那么我们就认为用户要使用标准输入流输入数据：

   ```c++
   	} else {
   		wc = wordcount(cin);
   	}	
   ```

6. 这样就完成了，然后我们只需要将统计出的单词数量保存在变量wc中即可：

   ```c++
   	cout << "There are " << wc << " words\n";
   };
   ```

7. 编译并运行程序。首先，我们从标准输入中进行输入。我们可以这里通过echo命令将字符串通过管道传递给程序。当然，我们也可以直接进行输入，并使用Ctrl+D来结束输入：

   ```c++
   $ echo "foo bar baz" | ./count_all_words
   There are 3 words
   ```

8. 这次我们使用文件作为输入源，并对其中单词数量进行统计：

   ```c++
   $ ./count_all_words count_all_words.cpp
   There are 61 words
   ```

## How it works...

本节也没有什么好多说的；实现很短，难度很低。需要提及的可能就是我们对std::cin和std::ifstream的实例进行了互换。cin是std::istream的类型之一，并且std::ifstream继承于std::istream。可以回顾一下本章开头的类型继承表。那么这两种类型即使在运行时，都是能进行互换的。

> Note：
>
> 使用流来保持代码的模块性。这有助于减少代码的耦合性，并且让代码更加容易测试，因为其可以匹配任意类型的流对象。