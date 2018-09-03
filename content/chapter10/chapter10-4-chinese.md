# 实现一个类似grep的文本搜索工具

大多数操作系统都会提供本地的搜索引擎。用户可以使用一些快捷键，对本地文件进行查找。

这种功能出现之前，命令行用户会通过`grep`或`awk`工具对文件进行查找。用户可以简单的输入` grep -r foobar . `，然后工具将会基于当前目录，进行递归的的查找，并显示包含有`"foobar"`名字的文件。

本节中，将实现这样一种应用。我们的`grep`使用命令行方式使用，并基于给定文件夹递归的对文件进行查找。然后，将找到的文件名打印出来。我们将使用线性的模式匹配方式，将匹配文件中的对应行号进行打印。

## How to do it...

我们将实现小工具，用于查找与用户提供的文本段匹配的文件。这工具与UNIX中的`grep`工具类似，不过为了简单起见，其功能没有那么强大：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <fstream>
   #include <regex>
   #include <vector>
   #include <string>
   #include <filesystem>
   
   using namespace std;
   using namespace filesystem;
   ```

2. 先来实现一个辅助函数，这个函数能接受一个文件地址和一个正则表达式对象，正则表达式对象用来描述我们要查找的文本段。然后，实例化一个`vector`，用于保存匹配的文件行和其对应的内容。然后，实例化一个输入文件流对象，读取文件，并进行逐行的文本匹配。

   ```c++
   static vector<pair<size_t, string>>
   matches(const path &p, const regex &re)
   {
       vector<pair<size_t, string>> d;
       ifstream is {p.c_str()};
   ```

3. 通过`getline`函数对文件进行逐行读取，当字符串中包含有我们提供文本段，则`regex_search`返回true，如果匹配会将字符串和对应的行号保存在`vector`中。最后，我们将返回所有匹配的结果：

   ```c++
       string s;
       for (size_t line {1}; getline(is, s); ++line) {
           if (regex_search(begin(s), end(s), re)) {
           	d.emplace_back(line, move(s));
           }
       }
   
       return d;
   }
   ```

4. 主函数会先对用户提供的文本段进行检查，如果这个文本段不能用，则返回错误信息：

   ```c++
   int main(int argc, char *argv[])
   {
       if (argc != 2) {
           cout << "Usage: " << argv[0] << " <pattern>\n";
           return 1;
       }
   ```

5. 接下来，会通过输入文本创建一个正则表达式对象。如果表达式是一个非法的正则表达式，这将会导致一个异常抛出。如果触发了异常，我们将对异常进行捕获并处理：

   ```c++
       regex pattern;
   
       try { pattern = regex{argv[1]}; }
       catch (const regex_error &e) {
           cout << "Invalid regular expression provided.n";
           return 1;
       }
   ```

6. 现在，可以对文件系统进行迭代，然后对我们提供的文本段进行匹配。使用`recursive_directory_iterator`对工作目录下的所有文件进行迭代。原理和之前章节的`directory_iterator`类似，不过会对子目录进行递归迭代。对于每个匹配的文件，我们都会调用辅助函数`matches`：

   ```c++
   	for (const auto &entry :
               recursive_directory_iterator{current_path()}) {
           auto ms (matches(entry.path(), pattern));
   ```

7. 如果有匹配的结果，我们将会对文件地址，对应文本行数和匹配行的内容进行打印：

   ```c++
       for (const auto &[number, content] : ms) {
           cout << entry.path().c_str() << ":" << number
           	 << " - " << content << '\n';
           }
       }
   }
   ```

8. 现在，准备一个文件`foobar.txt`，其中包含一些测试行：

   ```c++
   foo
   bar
   baz
   ```

9. 编译并运行程序，就会得到如下输出。我们在`/Users/tfc/testdir`文件夹下运行这个程序，我们先来对`bar`进行查找。在这个文件夹下，其会在`foobar.txt`的第二行和`testdir/dir1`文件夹下的另外一个文件`text1.txt`中匹配到：

   ```c++
   $ ./grepper bar
   /Users/tfc/testdir/dir1/text1.txt:1 - foo bar bla blubb
   /Users/tfc/testdir/foobar.txt:2 - bar
   ```

10. 再次运行程序，这次我们对`baz`进行查找，其会在第三行找到对应内容：

   ```c++
   $ ./grepper baz
   /Users/tfc/testdir/foobar.txt:3 - baz
   ```

##  How it works...

本节的主要任务是使用正则表达式对文件的内容进行查找。不过，让我们关注一下`recursive_directory_iterator`，因为我们会使用这个迭代器来进行本节的子文件夹的递归迭代。

与`directory_iterator`和`recursive_directory_iterator`迭代类似，其可以用来对子文件夹进行递归，就如其名字一样。当进入文件系统中的一个文件夹时，将会产生一个`directory_entry`实例。当递归到子文件夹时，也会产生对应的`directory_entry`实例。

`recursive_directory_iterator`具有一些有趣的成员函数：

- `depth()`代表我们需要迭代多少层子文件夹。
- `recursion_pending()`代表在进行当前迭代器后，是否会在进行对子文件夹进行迭代。
- `disable_recursion_pending()`当迭代器需要再对子文件夹进行迭代时，提前调用这个函数，则会让递归停止。
- `pop()`将会终止当前级别的迭代，并返回上一级目录。

## There's more...

我们需要了解的另一个就是`directory_options`枚举类。`recursive_directory_iterator`能将`directory_options`的实例作为其构造函数的第二个参数，通常将` directory_options::none`作为默认值传入。其他值为：

- `follow_directory_symlink`能允许对符号链接的文件夹进行递归迭代。
- `skip_permission_denied`这会告诉迭代器，是否跳过由于权限错误而无法访问的目录。

这两个选项可以通过`|`进行组合。