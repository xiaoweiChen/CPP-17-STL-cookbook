# 通过逻辑连接创建复杂谓词

当使用通用代码过滤数据时，我们通常会定义一些谓词，这些谓词就是告诉计算机，哪些数据是我们想要样，哪些数据时我们不想要的。通常谓词都是组合起来使用。

例如，当我们在过滤字符串时，我们需要实现一个谓词，当其发现输入的字符串以`foo`开头就返回true，其他情况都返回false。另一个谓词，当其发现输入的字符串以“bar”结尾时，返回true，否则返回false。

我们也不总是自己去定义谓词，有时候可以复用已经存在的谓词，并将它们结合起来使用。比如，如果我们既想要检查输入字符串的开头是否是`foo`，又想检查结尾是否为“bar”时，就可以将之前提到的两个谓词组合起来使用。本节我们使用Lambda表达式，用一种更加舒服的方式来完成这件事。

## How to do it...

我们将来实现一个非常简单的字符串过滤谓词，并且将其和辅助函数结合让其变得更加通用。

1. 包含必要的头文件

   ```c++
   #include <iostream>
   #include <functional>
   #include <string>
   #include <iterator>
   #include <algorithm> 
   ```

2. 这里实现两个简单的谓词函数，后面会用到它们。第一个谓词会告诉我们字符串的首字母是否是`a`，第二个谓词则会告诉我们字符串的结尾字母是否为`b`：

   ```c++
   static bool begins_with_a (const std::string &s)
   {
   	return s.find("a") == 0;
   }
   static bool ends_with_b (const std::string &s)
   {
   	return s.rfind("b") == s.length() - 1;
   }
   ```

3. 现在，让我们来实现辅助函数，我们称其为`combine`。其需要一个二元函数作为其第一个参数，可以是逻辑'与'或逻辑'或'操作。之后的两个参数为需要结合在一起的谓词函数：

   ```c++
   template <typename A, typename B, typename F>
   auto combine(F binary_func, A a, B b)
   {
   ```

4. 之后，我们会返回一个Lambda表达式，这个表达式可以获取到两个合并后的谓词。这个表达式需要一个参数，这个参数会传入两个谓词中，然后表达式将返回这个两个谓词结合后的结果：

   ```c++
       return [=](auto param) {
       	return binary_func(a(param), b(param));
       };
   }
   ```

5. 在实现主函数之前，先声明所使用命名空间：

   ```c++
   using namespace std;
   ```

6. 现在，让将两个谓词函数合并在一起，形成另一个全新的谓词函数，其会告诉我们输入的字符串是否以'a'开头，并且以'b'结尾，比如"ab"或"axxxb"就会返回true。二元函数我们选择`std::logical_and`。这是个模板类，需要进行实例化，所以这里我们使用大括号对创建其实例。需要注意的是，因为该类的默认类型为void，所以这里我们并没有提供模板参数。特化类的参数类型，都由编译器推导得到：

   ```c++
   int main()
   {
       auto a_xxx_b (combine(
           logical_and<>{},
           begins_with_a, ends_with_b));
   ```

7. 我们现在可以对标准输入进行遍历，然后打印出满足全新谓词的词组：

   ```c++
       copy_if(istream_iterator<string>{cin}, {},
               ostream_iterator<string>{cout, ", "},
               a_xxx_b);
       cout << '\n';
   } 
   ```

8. 编译边运行程序，就会得到如下输出。我们输入了四个单词，但是只有两个满足我们的谓词条件：

   ```c++
   $ echo "ac cb ab axxxb" | ./combine
   ab, axxxb,
   ```

##There's more...

STL已经提供了一些非常有用的函数对象，例如`std::logical_and`，`std::logical_or`等等。所以我们没有必要所有东西都自己去实现。可以去看一下C++的参考手册，了解一下都有哪些函数对象已经实现：

- 英文：http://en.cppreference.com/w/cpp/utility/functional
- 中文：http://zh.cppreference.com/w/cpp/utility/functional

