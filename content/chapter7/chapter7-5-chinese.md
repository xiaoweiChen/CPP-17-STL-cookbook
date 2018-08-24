# 计算文件中的单词数量

我们在读取一个文件的时候，也想知道这个文件中包含的单词数量。我们定义的单词是位于两个空格之间的字符组合。那要如何进行统计呢？

根据对单词的定义， 我们可以统计空格的数量。例如句子`John has a funny little dog.`，这里有五个空格，所以说这句话有六个单词。

如果句子中有空格干扰怎么办，例如：`   John   has    \t   a\nfunny little dog  .`。这句中有很多不必要的空格、制表符和换行符。本书的其他章节中，我们已经了解如何将多余空格从字符串中去掉。所以，可以对字符串进行预处理，将不必要的空格都去掉。这样做的确可行，不过我们有更加简单的方法。

为了寻找最优的解决方案，我们将让用户选择，是从标准输入中获取数据，还是从文本文件中获取数据。

## How to do it...

本节，我们将完成一个单行统计函数，其可以对输入的数据进行计数，数据源的具体方式我们可以让用户来选择。

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <fstream>
   #include <string>
   #include <algorithm>
   #include <iterator>

   using namespace std;
   ```

2. `wordcount`函数能接受一个输入流，例如`cin`。其能创建一个`std::input_iterator`迭代器，其能对输出字符进行标记，然后交由`std::distance`进行计算。`distance`接受两个迭代器作为参数，并确定从一个迭代器到另一个迭代器要用多少步(距离)。对于随机访问迭代器，因为有减法操作符的存在，所以实现起来非常简单。其迭代器如同指针一样，可以直接进行减法，计算出两点的距离。不过`istream_iterator`就不行，因为其是前向迭代器，只能向前读取，直至结束。最后所需要的步数也就是单词的数量：

   ```c++
   template <typename T>
   size_t wordcount(T &is)
   {
   	return distance(istream_iterator<string>{is}, {});
   }
   ```

3. 主函数中，我们会让用户来选择输入源：

   ```c++
   int main(int argc, char **argv)
   {
   	size_t wc;
   ```

4. 如果用户选择使用文件进行输入(例如：`./count_all_words some_textfile.txt`)，我们可以通过`argv`获取命令行中的文件名称，并将文件打开，读取数据，从而对其文本进行单词统计：

   ```c++
   	if (argc == 2) {
   		ifstream ifs {argv[1]};
   		wc = wordcount(ifs);
   ```

5. 如果用户没有传入任何参数，就认为用户要使用标准输入流输入数据：

   ```c++
   	} else {
   		wc = wordcount(cin);
   	}	
   ```

6. 然后只需要将统计出的单词数量保存在变量`wc`中即可：

   ```c++
   	cout << "There are " << wc << " words\n";
   };
   ```

7. 编译并运行程序。首先，从标准输入中进行输入。我们可以这里通过echo命令将字符串，通过管道传递给程序。当然，我们也可以直接进行输入，并使用`Ctrl+D`来结束输入：

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

本节也没有什么好多说的；实现很短，难度很低。需要提及的可能就是我们对`std::cin`和`std::ifstream`的实例进行了互换。`cin`是`std::istream`的类型之一，并且`std::ifstream`继承于`std::istream`。可以回顾一下本章开头的类型继承表。这两种类型即使在运行时，都能进行互换。

> Note：
>
> 使用流来保持代码的模块性，这有助于减少代码的耦合性。因为其可以匹配任意类型的流对象，所以更容易对代码进行测试。