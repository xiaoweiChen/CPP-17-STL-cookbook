# 实现标准化路径

本节中我们通过一个非常简单的例子来了解`std::filesystem::path`类，并实现一个智能标准化系统路径的辅助函数。

本节中的例子可以在任意的文件系统中使用，并且返回一种标准化格式的路径。标准化就意味着获取的是绝对路径，路径中不包括`.`和`..`。

实现函数的时候，我们将会了解，当使用文件系统库的基础部分时，需要注意哪些细节。

## How to do it...

本节中，我们的程序可以从命令行参数中获得文件系统路径，并使用标准化格式进行打印：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <filesystem>
   
   using namespace std;
   using namespace filesystem;
   ```

2. 主函数中，会对命令行传入的参数进行检查。如果没有传入，我们将会直接返回，并在终端上打印程序具体的使用方式。当提供了一个路径，那我们将用其对`filesystem::path`对象进行实例化：

   ```c++
   int main(int argc, char *argv[])
   {
       if (argc != 2) {
           cout << "Usage: " << argv[0] << " <path>\n";
           return 1;
       }
       
       const path dir {argv[1]};
   ```

3. 实例化`path`对象之后，还不能确定这个路径是否真实存在于计算机的文件系统中。这里我们使用了`  filesystem::exists`来确认路径。如果路径不存在，我们会再次返回：

   ```c++
   	if (!exists(dir)) {
           cout << "Path " << dir << " does not exist.\n";
           return 1;
       }	
   ```

4. Okay，如果完成了这个检查，我们就能确定这是一个正确的路径，并且将会对这个路径进行标准化，然后将其进行打印。` filesystem::canonical`将会为我们返回另一个`path`对象，可以直接对其进行打印，不过`path`的`<<`重载版本会将双引号进行打印。为了去掉双引号，我们通过`.c_str()`或`.string()`方法对路径进行打印：

   ```c++
   	cout << canonical(dir).c_str() << '\n';
   }
   ```

5. 编译代码并运行。当我们在家目录下输入相对地址`"src"`时，程序将会打印出其绝对路径：

   ```c++
   $ ./normalizer src
   /Users/tfc/src
   ```

6. 当我们打印一些更复杂的路径时，比如：给定路径中包含桌面文件夹的路径，`..`，还会有`Documents`文件夹，然后在到`src`文件夹。然而，程序会打印出与上次相同的地址！

   ```c++
   $ ./normalizer Desktop/../Documents/../src
   /Users/tfc/src
   ```

## How it works...

作为一个`std::filesystem `的新手，看本节的代码应该也没有什么问题。通过文件系统路径字符串初始化了一个`path`对象。`std::filesystem::path`类为文件系统库的核心，因为库中大多数函数和类与之相关。

`filesystem::exists`函数可以用来检查给定的地址是否存在。检查文件路径的原因是，`path`对象中的地址，不确定在文件系统中是否存在。`exists`能够接受一个`path`实例，如果地址存在，则返回`true`。`exists`无论是相对地址和绝对地址都能够进行判断。

最后，我们使用了`filesystem::canonical`将给定路径进行标准化。

```c++
path canonical(const path& p, const path& base = current_path());
```

`canonical`函数能接受一个`path`对象和一个可选的第二参数，也就是另一个地址。如果`p`路径是一个相对路径，那么`base`就是其基础路径。完成这些后，`canonical`会将`.`和`..`从路径中移除。

打印时对标准化地址使用了`.c_str()`函数，这样我们打印出来的地址前后就没有双引号了。

## There's more...

`canonical`在对应地址不存在时，会抛出一个`filesystem_error`类型的异常。为了避免函数抛出异常，我们需要使用`exists`函数对提供路径进行检查。这样的检查仅仅就是为了避免函数抛出异常吗？肯定不是。

`exists` 和`canonical`函数都能抛出`bad_alloc`异常。如果我们遇到了，那程序肯定会失败。更为重要的是，当我们对路径进行标准化处理时，其他人将对应的文件重命名或删除了，则会造成更严重的问题。这样的话，即便是之前进行过检查，`canonical`还是会抛出一个`filesystem_error`异常。

大多数系统函数都会有一些重载，他们能够接受相同的参数，甚至是一个` std::error_code`引用：

```c++
path canonical(const path& p, const path& base = current_path());
path canonical(const path& p, error_code& ec);
path canonical(const std::filesystem::path& p,
               const std::filesystem::path& base,
               std::error_code& ec );
```

我们可以使用`try-catch`将系统函数进行包围，手动的对其抛出的异常进行处理。需要注意的是，这里只会改变系统相关错误的动作，而无法对其他进行修改。带或不带`ec`参数，更加基于异常，例如当系统没有可分配内存时，还是会抛出`bad_alloc`异常。