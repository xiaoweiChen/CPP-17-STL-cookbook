# 计算文件类型的统计信息

上一节中，我们实现了一个用于统计任意文件夹中所有文件大小的工具。

本节中，我们将递归的对文件夹中的文件名后缀进行统计。这样对每种文件类型的文件进行个数统计，并且计算每种文件类型大小的平均值。

## How to do it...

本节中将实现一个简单的工具用于对给定的文件夹进行递归，同时对所有文件的数量和大小进行统计，并通过文件后缀进行分组。最后，会对文件夹中具有的文件名扩展进行打印，并打印出有多少个对应类型扩展的文件和文件的平均大小。

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <sstream>
   #include <iomanip>
   #include <map>
   #include <filesystem>
   
   using namespace std;
   using namespace filesystem;
   ```

2. `size_string`函数已经在上一节中使用过了。这里我们继续使用：

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

3. 然后，实现一个函数用于接受一个`path`对象，并对该路径下的所有文件进行遍历。我们使用一个`map`来收集所有的信息，用对应的扩展名与总体数量和所有文件的总大小进行统计：

   ```c++
   static map<string, pair<size_t, size_t>> ext_stats(const path &dir)
   {
       map<string, pair<size_t, size_t>> m;
       
       for (const auto &entry :
       	recursive_directory_iterator{dir}) {	
   ```

4. 如果目录入口是一个目录，我们将跳过这个入口。跳过的意思就是不会对这个目录进行递归操作。`recursive_directory_iterator`可以完成这个工作，但是不需要去查找所有文件夹中的文件。

   ```c++
           const path p {entry.path()};
           const file_status fs {status(p)};
   
           if (is_directory(fs)) { continue; }		
   ```

5. 接下来，会对文件的扩展名进行提取。如果文件没有扩展名，就会对其进行忽略：

   ```c++
   		const string ext {p.extension().string()};
   
   		if (ext.length() == 0) { continue; }
   ```

6. 接着，计算我们查找到文件的总体大小。然后，将会对`map`中同一扩展的对象进行聚合。如果对应类型还不存在，创建起来也很容易。我们可以简单的对文件计数进行增加，并且对扩展总体大小进行累加：

   ```c++
           const size_t size {file_size(p)};
   
           auto &[size_accum, count] = m[ext];
   
           size_accum += size;
           count += 1;
       }
   ```

7. 之后，我们会返回这个`map`：

   ```c++
   	return m;
   }
   ```

8. 主函数中，我们会从用户提供的路径中获取对应的路径，或是使用当前路径。当然，需要对地址是否存在进行检查，否则继续下去就没有任何意义：

   ```c++
   int main(int argc, char *argv[])
   {
       path dir {argc > 1 ? argv[1] : "."};
       
       if (!exists(dir)) {
           cout << "Path " << dir << " does not exist.\n";
           return 1;
       }
   ```

9. 可以对`ext_stats`进行遍历。因为`map`中的`accum_size`元素包含有同类型扩展文件的总大小，然后用其除以总数量，以计算出平均值：

   ```c++
       for (const auto &[ext, stats] : ext_stats(dir)) {
           const auto &[accum_size, count] = stats;
           
           cout << setw(15) << left << ext << ": "
                << setw(4) << right << count
                << " items, avg size "
                << setw(4) << size_string(accum_size / count)
                << '\n';
       }
   }
   ```

10. 编译并运行程序，我们将会得到如下的输出。我将C++离线手册的地址，作为命令行的参数：

  ```c++
  $ ./file_type ~/Documents/cpp_reference/
  .css :2 items, avg size 41K
  .gif :7 items, avg size 902B
  .html: 4355 items, avg size 38K
  .js:3 items, avg size 4K
  .php :1 items, avg size 739B
  .png : 34 items, avg size 2K
  .svg : 53 items, avg size 6K
  .ttf :2 items, avg size 421K
  ```


