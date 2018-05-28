# 消除字符串开始和结束处的空格

应用从用户端获取到的输入，经常会有很多不必要的空格存在。在之前的章节中，我们将单词间多余的空格进行移除。

现在让我们来看看被空格包围的字符串应该怎么去移除多余的空格。std::string具有很多不错的辅助函数能完成这项工作。

> Note：
>
> 这节看完后，下节也别错过。将会在下节看到我们如何使用std::string_view类来避免不必要的拷贝或数据修改。

## How to do it...

本节中，我们将完成一个辅助函数的实现，其将判断是否有多余的空格在字符串开头和结尾，并复制返回去掉这些空格的字符串，并进行简单的测试：

1. 包含必要的头文件，并声明所使用的命名空间。

   ```c++
   #include <iostream>
   #include <string>
   #include <algorithm>
   #include <cctype>

   using namespace std;
   ```

2. 我们的函数将对一个常量字符串进行首尾空格的去除。并返回首尾没有空格的新字符串：

   ```c++
   string trim_whitespace_surrounding(const string &s)
   { 
   ```

3. std::string能够提供两个函数，这两个函数对我们很有帮助。第一个就是string::find_first_not_of，其能帮助我们找到我们想要跳过的字符。本节中毫无疑问就是空格，其包括空格' '，制表符'\t'和换行符'\n'。函数能返回第一个非空格字符的位置。如果字符串里面只有空格，那么其间返回string::npos。这意味着我们没有找到除了空格的其他字符。如果这样，我们就会返回一个空的字符串：

   ```c++
   	const char whitespace[] {" \t\n"};
   	const size_t first (s.find_first_not_of(whitespace));
   	if (string::npos == first) { return {}; }
   ```

4. 现在我们知道新字符串从哪里开始，但是再哪里结尾呢？因此，我们需要使用另一个函数string::find_last_not_of。其能帮助我们找到最后一个非空格字符的位置：

   ```c++
   	const size_t last (s.find_last_not_of(whitespace));
   ```

5. 使用string::substr就能返回子字符串，返回的字符串没有空格。这个函数需要两个参数——一个是字符串的起始位置，另一个是字符串的长度：

   ```c++
   	return s.substr(first, (last - first + 1));
   }
   ```

6. 这就完成了。现在让我们来编写主函数，创建字符串，让字符串的前后充满空格，以便我们进行移除：

   ```c++
   int main()
   {
       string s {" \t\n string surrounded by ugly"
       		 " whitespace \t\n "};
   ```

7. 我们将打印去除前和去除后的字符串。将字符串放入大括号中，这样就很容易辨别哪里有空格了：

   ```c++
       cout << "{" << s << "}\n";
       cout << "{"
       	 << trim_whitespace_surrounding(s)
       	 << "}\n";
   }
   ```

8. 编译运行程序，就会得到如下的结果：

   ```c++
   $ ./trim_whitespace
   {
   string surrounded by ugly whitespace
   }
   {string surrounded by ugly whitespace}
   ```

## How it works...

本节中，我们使用了string::find_first_not_of和string::find_last_not_of函数。这两个函数也能接受C风格的字符串，会将其当做字符链表进行搜索。当我们有一个字符串"foo bar"时，当我们调用`find_first_not_of("bfo ")`时，去将返回5，因为'a'字符是第一个不属于"bfo "的字符。参数中字符的顺序，在这里并不重要。

倒装的函数也是同样的原理，当然还有两个我们没有在本节使用到的函数：string::find_first_of和string::find_last_of。

同样也是基于迭代器的函数，我么需要检查函数是否返回了合理的位置，当没有找到时，函数会返回一个特殊的位置——string::npos。

我们可以从辅助函数中找出字符所在的位置，并且使用string::substr来构造前后没有空格的字符串。这个函数接受一个首字符相对位置和字符串长度，然后就会构造一个子字符串进行返回。举个栗子，`string{"abcdef"}.substr(2, 2)`将返回`"cd"`。