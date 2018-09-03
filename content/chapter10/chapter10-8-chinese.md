# 实现一个工具：通过符号链接减少重复文件，从而控制文件夹大小

很多工具以不同的方式对数据进行压缩。其中最著名的文件压缩算法/格式就是ZIP和RAR。这种工具通过减少文件内部冗余，从而减少文件的大小。

将文件压缩成压缩包外，另一个非常简单的减少磁盘使用率的范式就是删除重复的文件。本节中，我们将实现一个小工具，其会对目录进行递归。递归中将对文件内容进行对比，如果找到相同的文件，我们将对其中一个进行删除。所有删除的文件则由一个符号链接替代，该链接指向目前唯一存在的文件。这种方式可以不通过压缩对空间进行节省，同时对所有的数据能够进行保存。

## How to do it...

本节中，将实现一个小工具用来查找那些重复的文件。我们将会删除其中一个重复的文件，并使用符号链接的方式对其进行替换，这样就能减小文件夹的大小。

> Note：
>
> 为了对系统数据进行备份，我们将使用STL函数对文件进行删除。一个简单的拼写错误就可能会删除很多并不想删除的文件。

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <fstream>
   #include <unordered_map>
   #include <filesystem>
   
   using namespace std;
   using namespace filesystem;
   ```

2. 为了查找重复的文件，我们将构造一个哈希表，并对将文件哈希值与其第一次产生的地址相对应。最好的方式就是通过哈希算法，对文件计算出一个MD5或SHA码。为了保证例子的简洁，我们将会把文件读入一个字符串中，然后使用`hash`函数计算出对应的哈希值：

   ```c++
   static size_t hash_from_path(const path &p)
   {
       ifstream is {p.c_str(),
       	ios::in | ios::binary};
       if (!is) { throw errno; }
       
       string s;
       
       is.seekg(0, ios::end);
       s.reserve(is.tellg());
       is.seekg(0, ios::beg);
       
       s.assign(istreambuf_iterator<char>{is}, {});
       
       return hash<string>{}(s);
   }
   ```

3. 然后，我们会实现一个哈希表，并且删除重复的文件。其会对当前文件夹和其子文件夹进行遍历：

   ```c++
   static size_t reduce_dupes(const path &dir)
   {
       unordered_map<size_t, path> m;
       size_t count {0};
       
       for (const auto &entry :
       	 recursive_directory_iterator{dir}) { 
   ```

4. 对于每个文件入口，我们都会进行检查，当其是文件夹时就会跳过。对于每一个文件，我们都会产生一个哈希值，并且尝试将其插入哈希表中。当哈希表已经包含有相同的哈希值，这也就意味着有文件重复了。并且插入操作会终止，`try_emplace`所返回的第二个值就是false:

   ```c++
       const path p {entry.path()};
   
       if (is_directory(p)) { continue; }
   
       const auto &[it, success] =
           m.try_emplace(hash_from_path(p), p);
   ```

5. `try_emplace`的返回值将告诉我们，该键是否是第一次插入的。这样我们就能找到重复的，并告诉用户文件有重复的，并将重复的进行删除。删除之后，我们将为重复的文件创建符号链接：

   ```c++
       if (!success) {
           cout << "Removed " << p.c_str()
                << " because it is a duplicate of "
                << it->second.c_str() << '\n';
           
           remove(p);
           create_symlink(absolute(it->second), p);
           ++count;
       }	
   ```

6. 对文件系统进行插入后，我们将会返回重复文件的数量：

   ```c++
   	}
   
   	return count;
   }
   ```

7. 主函数中，我们会对用户在命令行中提供的目录进行检查。

   ```c++
   int main(int argc, char *argv[])
   {
       if (argc != 2) {
           cout << "Usage: " << argv[0] << " <path>\n";
           return 1;
       }
       
       path dir {argv[1]};
       
       if (!exists(dir)) {
           cout << "Path " << dir << " does not exist.\n";
           return 1;
       }
   ```

8. 现在我们只需要对`reduce_dupes`进行调用，并打印出有多少文件被删除了：

   ```c++
       const size_t dupes {reduce_dupes(dir)};
   
       cout << "Removed " << dupes << " duplicates.\n";
   }
   ```

9. 编译并运行程序，输出中有一些看起来比较复杂的文件。程序执行之后，我会使用`du`工具来检查文件夹的大小，并证明这种方法是有效的。

   ```c++
   $ du -sh dupe_dir
   1.1Mdupe_dir
   
   $ ./dupe_compress dupe_dir
   Removed dupe_dir/dir2/bar.jpg because it is a duplicate of
   dupe_dir/dir1/bar.jpg
   Removed dupe_dir/dir2/base10.png because it is a duplicate of
   dupe_dir/dir1/base10.png
   Removed dupe_dir/dir2/baz.jpeg because it is a duplicate of
   dupe_dir/dir1/baz.jpeg
   Removed dupe_dir/dir2/feed_fish.jpg because it is a duplicate of
   dupe_dir/dir1/feed_fish.jpg
   Removed dupe_dir/dir2/foo.jpg because it is a duplicate of
   dupe_dir/dir1/foo.jpg
   Removed dupe_dir/dir2/fox.jpg because it is a duplicate of
   dupe_dir/dir1/fox.jpg
   Removed 6 duplicates.
       
   $ du -sh dupe_dir
   584Kdupe_dir
   ```

## How it works...

使用`create_symlink`函数在文件系统中链接一个文件，指向另一个地方。这样就能避免重复的文件出现，也可以使用`create_hard_link`设置一些硬链接。硬链接和软连接相比，有不同的技术含义。有些格式的文件系统可能不支持硬链接，或者是使用一定数量的硬链接，指向相同的文件。另一个问题就是，硬链接没有办法让两个文件系统进行链接。

不过，除开实现细节，使用`create_symlink`或`create_hard_link`时，会出现一个明显的错误。下面的几行代码中就有一个bug。你能很快的找到它吗？

```c++
path a {"some_dir/some_file.txt"};
path b {"other_dir/other_file.txt"};
remove(b);
create_symlink(a, b);
```

在程序执行的时候，不会发生任何问题，不过符号链接将失效。符号链接将错误的指向`some_dir/some_file.txt`。正确指向的地址应该是`/absolute/path/some_dir/some_file.txt`或`../some_dir/some_file.txt`。`create_symlink`使用正确的绝对地址，可以使用如下写法：

 ```c++
create_symlink(absolute(a), b);
 ```

> Note：
>
> `create_symlink`不会对链接进行检查

## There's more...

可以看到，哈希函数非常简单。为了让程序没有多余的依赖，我们采用了这种方式。

我们的哈希函数有什么问题呢？有两个问题：

- 会将一个文件完全读入到字符串中。如果对于很大的文件来说，这将是一场灾难。
- C++ 中的哈希函数`  hash<string>`可能不是这样使用的。

要寻找一个更好的哈希函数时，我们需要找一个快速、内存友好、简单的，并且保证不同的文件具有不同的哈希值，最后一个需求可能是最关键的。因为我们使用哈希值了判断两个文件是否一致，当我们认为两个文件一致时，但哈希值不一样，就能肯定有数据受到了损失。

比较好的哈希算法有MD5和SHA(有变体)。为了让我们程序使用这样的函数，可能需要使用OpenSSL中的密码学API。

