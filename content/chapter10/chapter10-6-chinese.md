# 实现一个磁盘使用统计器

我们已经实现了一个列出文件夹下所有文件的工具，不过和系统自带的工具一样，其都不会对文件夹的大小进行打印。

为了获取文件夹的大小，我们需要将其子文件夹进行迭代，然后将其包含的所有文件的大小进行累加，才能得到该文件夹的大小。

本节中，我们将实现一个工具用来做这件事。这个工具能在任意的文件夹下运行，并且对文件夹中包含的文件总体大小进行统计。

## How to do it...

本节中，我们将会实现一个程序用于迭代目录中的所有文件，并将所有文件的大小进行统计。对于统计一个文件的大小就很简单，但是要统计一个文件夹的大小，就需要将文件夹下的所有文件的大小进行相加。

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <sstream>
   #include <iomanip>
   #include <numeric>
   #include <filesystem>
   
   using namespace std;
   using namespace filesystem;
   ```

2. 我们将实现一个辅助函数使用`directory_entry`对象作为其参数，然后返回其在文件系统中对应的大小。如果传入的不是一个文件夹地址，将通过`file_size`获得文件的大小。

   ```c++
   static size_t entry_size(const directory_entry &entry)
   {
   	if (!is_directory(entry)) { return file_size(entry); }
   ```

3. 如果传入的是一个文件夹，需要对其中所有元素进行文件大小的计算。需要调用辅助函数`entry_size`对子文件夹进行再次递归：

   ```c++
       return accumulate(directory_iterator{entry}, {}, 0u,
           [](size_t accum, const directory_entry &e) {
           	return accum + entry_size(e);
           });
   }
   ```

4. 为了具有更好的可读性，本节使用了其他章节中的`size_string`函数。

   ```c++
   static string size_string(size_t size)
   {
       stringstream ss;
       if (size >= 1000000000) {
       	ss << (size / 1000000000) << 'G';
       } else if (size >= 1000000) {
       	ss << (size / 1000000) << 'M';
       } else if (size >= 1000) {
       	ss << (size / 1000) << 'K';
       } else { ss << size << 'B'; }
       
       return ss.str();
   }
   ```

5. 主函数中，首先就是要检查用户通过命令行提供的文件系统路径。如果没有提供，则默认为当前文件夹。处理之前，我们会检查路径是否存在。

   ```c++
   int main(int argc, char *argv[])
   {
       path dir {argc > 1 ? argv[1] : "."};
       
       if (!exists(dir)) {
       cout << "Path " << dir << " does not exist.\n";
       	return 1;
       } 
   ```

6. 现在，我们可以对所有的文件夹进行迭代，然后打印其名字和大小：

   ```c++
       for (const auto &entry : directory_iterator{dir}) {
           cout << setw(5) << right
                << size_string(entry_size(entry))
                << " " << entry.path().filename().c_str()
                << '\n';
       }
   }
   ```

7. 编译并运行程序，我们将获得如下的输出。我提供了一个C++离线手册的目录，其当然具有子目录，我们可以用我们的程序对其大小进行统计：

   ```c++
   $ ./file_size ~/Documents/cpp_reference/en/
   19M c
   12K c.html
   147M cpp
   17K cpp.html
   22K index.html
   22K Main_Page.html
   ```

## How it works...

整个程序通过`file_size`对普通的文件进行大小的统计。当程序遇到一个文件夹，其将会对子文件夹进行递归，然后通过`file_size`计算出文件夹中包含所有文件的大小。

有件事我们需要区别一下，当我们直接调用`file_size`时，或需要进行递归时，需要通过`is_directory`谓词进行判断。这对于只包含有普通文件和文件夹的文件夹是有用的。

与我们的简单程序一样，程序会在如下的情况下崩溃，因为有未处理的异常抛出：

- `file_size`只能对普通文件和符号链接有效。否则，会抛出异常。
- `file_size`对符号链接有效，如果链接失效，函数还是会抛出异常。

为了让本节的程序更加成熟，我们需要更多的防御性编程，避免遇到错误的文件和手动处理异常。

