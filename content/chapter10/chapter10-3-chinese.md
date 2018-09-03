# 列出目录下的所有文件

每个操作系统都会提供一些工具，以列出目录下的所有文件。Linux，MacOS和类UNIX的操作系统中，`ls`就是一个最简单的例子。Windows和Dos系统下，命令为`dir`。其会提供一些文件的补充信息，比如文件大小，访问权限等。

可以通过对文件夹的递归和文件遍历来对这样的工具进行实现。所以，让我们来试一下吧！

我们的`ls/dir`命令会将目录下的文件名，元素索引，以及一些访问权限标识，以及对应文件的文件大小，分别进行展示。

## How to do it...

本节中，我们将实现一个很小的工具，为使用者列出对应文件夹下的所有文件。会将文件名，文件类型，大小和访问权限分别列出来。

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <sstream>
   #include <iomanip>
   #include <numeric>
   #include <algorithm>
   #include <vector>
   #include <filesystem>
   
   using namespace std;
   using namespace filesystem;
   ```

2. `file_info`是我们要实现的一个辅助函数。其能接受一个`directory_entry`对象的引用，并从这个路径中提取相应的信息，实例化`file_status`对象(使用`status`函数)，其会包含文件类型和权限信息。最后，如果是一个常规文件，则会提取其文件大小。对于文件夹或一些特殊的文件，我们将返回大小设置为0。所有的信息都将会封装到一个元组中：

   ```c++
   static tuple<path, file_status, size_t>
   file_info(const directory_entry &entry)
   {
       const auto fs (status(entry));
       return {entry.path(),
               fs,
               is_regular_file(fs) ? file_size(entry.path()) : 0u};
   }
   ```

3. 另一个辅助函数就是`type_char`。路径不能仅表示目录和简单文本/二进制文件。操作系统提供了多种抽象类型，比如字符/块形式的硬件设备接口。STL库也提供了为此提供了很多为此函数。我们通过返回`'d'`表示文件夹，通过返回`'f'`表示普通文件等。

   ```c++
   static char type_char(file_status fs)
   {
       if (is_directory(fs)) { return 'd'; }
       else if (is_symlink(fs)) { return 'l'; }
       else if (is_character_file(fs)) { return 'c'; }
       else if (is_block_file(fs)) { return 'b'; }
       else if (is_fifo(fs)) { return 'p'; }
       else if (is_socket(fs)) { return 's'; }
       else if (is_other(fs)) { return 'o'; }
       else if (is_regular_file(fs)) { return 'f'; }
       
       return '?';
   }
   ```

4. 下一个辅助函数为`rwx`。其能接受一个`perms`变量(其为文件系统库的一个`enum`类)，并且会返回一个字符串，比如`rwxrwxrwx`，用来表示文件的权限设置。`"rwx" `分别为**r**ead, **w**rite和e**x**ecution，分别代表了文件的权限属性。每三个字符表示一个组，也就代表对应的组或成员，能对文件进行的操作。`rwxrwxrwx`则代表着每个人多能对这个文件进行访问和修改。`rw-r--r--`代表着所有者可以的对文件进行读取和修改，不过其他人只能对其进行读取操作。我们将这些`读取/修改/执行`所代表的字母进行组合，就能形成文件的访问权限列表。Lambda表达式可以帮助我们完成重复性的检查工作，检查`perms`变量`p`中是否包含特定的掩码位，然后返回`'-'`或正确的字符。

   ```c++
   static string rwx(perms p)
   {
       auto check ([p](perms bit, char c) {
       	return (p & bit) == perms::none ? '-' : c;
       });
       return {check(perms::owner_read, 'r'),
               check(perms::owner_write, 'w'),
               check(perms::owner_exec, 'x'),
               check(perms::group_read, 'r'),
               check(perms::group_write, 'w'),
               check(perms::group_exec, 'x'),
               check(perms::others_read, 'r'),
               check(perms::others_write, 'w'),
               check(perms::others_exec, 'x')};
   }
   ```

5. 最后一个辅助函数能接受一个整型的文件大小，并将其转换为跟容易读懂的模式。将其大小除以表示的对应边界，然后使用K, M或G来表示这个文件的大小：

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

6. 现在来实现主函数。我们会对用户在命令行输入的路径进行检查。如果没有传入，则默认为当前路径。然后，再来检查文件夹是否存在。如果不存在，就不会列出任何文件：

   ```c++
   int main(int argc, char *argv[])
   {
       path dir {argc > 1 ? argv[1] : "."};
       
       if (!exists(dir)) {
           cout << "Path " << dir << " does not exist.\n";
           return 1;
       }
   ```

7. 现在，将使用文件信息元组来填充一个`vector`。实例化一个`directory_iterator`，并且将其传入`path`对象的构造函数中。并通过目录迭代器对文件进行迭代，我们将`directory_entry`对象转换成文件信息元组，然后将其插入相应的`vector`。

   ```c++
       vector<tuple<path, file_status, size_t>> items;
       
       transform(directory_iterator{dir}, {},
       	back_inserter(items), file_info);	
   ```

8. 现在，将所有文件的信息都存在于`vector`之中，并且使用辅助函数将其进行打印：

   ```c++
       for (const auto &[path, status, size] : items) {
           cout << type_char(status)
                << rwx(status.permissions()) << " "
                << setw(4) << right << size_string(size)
                << " " << path.filename().c_str()
                << '\n';
   	}
   }
   ```

9. 编译并运行程序，并通过命令行传入C++文档文件所在的地址。我们能了解到对应文件夹所包含的文件，因为文件夹下只有`'d'`和`'f'`作为输出的表示。这些文件具有不同的权限，并且都有不同的大小。需要注意的是，这些文件的显示顺序，是按照名字在字母表中的顺序排序，不过我们不依赖这个顺序，因为C++17标准不需要字母表排序：

   ```c++
   $ ./list ~/Documents/cpp_reference/en/cpp
   drwxrwxr-x 0B   algorithm
   frw-r--r-- 88K  algorithm.html
   drwxrwxr-x 0B   atomic
   frw-r--r-- 35K  atomic.html
   drwxrwxr-x 0B   chrono
   frw-r--r-- 34K  chrono.html
   frw-r--r-- 21K  comment.html
   frw-r--r-- 21K  comments.html
   frw-r--r-- 220K compiler_support.html
   drwxrwxr-x 0B   concept
   frw-r--r-- 67K  concept.html
   drwxr-xr-x 0B   container
   frw-r--r-- 285K container.html
   drwxrwxr-x 0B   error
   frw-r--r-- 52K  error.html
   ```

## How it works...

本节中，我们迭代了文件夹中的所有文件，并且对每个文件的状态和大小进行检查。对于每个文件的操作都非常直接和简单，我们对文件夹的遍历看起来也很魔幻。

为了对我们的文件夹进行遍历，只是对`directory_iterator`进行实例化，然后对该对象进行遍历。使用文件系统库来遍历一个文件夹是非常简单的。

```c++
for (const directory_entry &e : directory_iterator{dir}) {
	// do something
}
```

除了以下几点，`directory_iterator`也没有什么特别的：

- 会对文件夹中的每个文件访问一次
- 文件中元素的遍历顺序未指定
- 文件节元素中`.`和`..`都已经被过滤掉

不过，`directory_iterator`看起来是一个迭代器，并且同时具有一个可迭代的范围。为什么需要注意这个呢？对于简单的`for`循环来说，其需要一个可迭代的范围。本节例程中，我们会将其当做一个迭代器使用：

```c++
transform(directory_iterator{dir}, {},
		 back_inserter(items), file_info);
```

实际上，就是一个迭代器类型，只不过这个类将`std::begin`和`std::end`函数进行了重载。当调用`begin`和`end`时，其会返回相应的迭代器。虽说第一眼看起来比较奇怪，但是让这个类型的确更加有用。