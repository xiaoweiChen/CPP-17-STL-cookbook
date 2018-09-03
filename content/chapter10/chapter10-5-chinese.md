# 实现一个自动文件重命名器

本节的动机是因为我自己经常需要使用到这样的功能。我们将假日中的照片汇总在一起时，不同朋友的照片和视频都在一个文件夹中，并且每个文件的后缀看起来都不一样。一些JPEG文件有`.jpg`的扩展，而另一些为`.jpeg`，还有一些则为`.JPEG`。

一些人会让文件具有统一的扩展，其会使用一些有用的命令对于所有文件进行重命名。同时，我们会将使用下划线来替代空格。

本节中，我们将试下一个类似的工具，叫做`renamer`。其能接受一些列输入文本段，作为其替代，类似如下的方式：

```c++
$ renamer jpeg jpg JPEG jpg
```

本节中，重命名器将会对当前目录进行递归，然后找到文件后缀为`jpeg`和`JPEG`的所有文件，并将这些文件的后缀统一为`jpg`。

##  How to do it...

我们将实现一个工具，通过对文件夹的递归对于所有文件名匹配的文件进行重命名。所有匹配到的文件，都会使用用户提供的文本段进行替换。

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <regex>
   #include <vector>
   #include <filesystem>
   
   using namespace std;
   using namespace filesystem;
   ```

2. 我们将实现一个简单的辅助函数，其能接受一个使用字符串表示的输入文件地址和一组替换对。每一个替换对都有一个文本段和其要替换文本段。对替换范围进行循环时，我们使用了`regex_replace`用于对输入字符串进行匹配，然后返回转换后的字符串。之后，我们将返回结果字符串。

   ```c++
   template <typename T>
   static string replace(string s, const T &replacements)
   {
       for (const auto &[pattern, repl] : replacements) {
       	s = regex_replace(s, pattern, repl);
       }
       
       return s;
   }
   ```

3. 主函数中，我们首先对命令行的正确性进行检查。可以成对的接受命令行参数，因为我们想要匹配段和替换段相对应。`argv`的第一个元素为执行文件的名字。当用户提供了成对的匹配段和替换段时，`argc`肯定是大于3的奇数：

   ```c++
   int main(int argc, char *argv[])
   {
       if (argc < 3 || argc % 2 != 1) {
           cout << "Usage: " << argv[0]
           	 << " <pattern> <replacement> ...\n";
           return 1;
       }
   ```

4. 我们对输入对进行检查时，会将对应的`vector`进行填充：

   ```c++
       vector<pair<regex, string>> patterns;
   
       for (int i {1}; i < argc; i += 2) {
       	patterns.emplace_back(argv[i], argv[i + 1]);
       }
   ```

5. 现在，可以对整个文件系统进行遍历。简单起见，将当前目录作为遍历的默认起始地址。对于每一个文件夹入口，我们将其原始路径命名为`opath`。然后，只在没有剩余路径的情况下使用文件名，并根据之前创建的匹配列表，对对应的匹配段进行替换。我们将拷贝`opath`到`rpath`中，并且将文件名进行替换。

   ```c++
       for (const auto &entry :
       		recursive_directory_iterator{current_path()}) {
      		 		path opath {entry.path()};
       			string rname {replace(opath.filename().string(),
       			patterns)};
           
           path rpath {opath};
           rpath.replace_filename(rname);
   ```

6. 对于匹配的文件，我们将打印其重命名后的名字。当重命名后的文件存在，我们将不会对其进行处理。会跳过这个文件。当然，我们也可以添加一些数字或其他字符到地址中，从而解决这个问题：

   ```c++
           if (opath != rpath) {
               cout << opath.c_str() << " --> "
               	<< rpath.filename().c_str() << '\n';
               if (exists(rpath)) {
               	cout << "Error: Can't rename."
               		" Destination file exists.\n";
               } else {
               	rename(opath, rpath);
               }
           }
       }
   }
   ```

7. 编译并运行程序，我们将会得到如下的输出。我的文件夹下面有一些JPEG文件，但是都是以不同的后缀名结尾，有`jpg`，`jpeg`和`JPEG`。然后，执行程序将`jpeg`和`JPEG`替换成`jpg`。这样，就可以对文件名进行统一化。

   ```c++
   $ ls
   birthday_party.jpeg holiday_in_dubai.jpgholiday_in_spain.jpg
   trip_to_new_york.JPEG
   $ ../renamer jpeg jpg JPEG jpg
   /Users/tfc/pictures/birthday_party.jpeg --> birthday_party.jpg
   /Users/tfc/pictures/trip_to_new_york.JPEG --> trip_to_new_york.jpg
   $ ls
   birthday_party.jpg holiday_in_dubai.jpg holiday_in_spain.jpg
   trip_to_new_york.jpg
   ```
