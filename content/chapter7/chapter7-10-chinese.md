# 使用特定代码段将输出重定向到文件

`std::cout`为我们提供了一种非常方便的打印方式，使用起来也十分方便，易于扩展，并可全局访问。即使我们想打印对应的信息时，比如错误信息，我们可以使用错误输出`std::cerr`进行输出，其和`cout`的用法一样，只不过一个从标准通道进行输出，另一个从错误通道进行输出。

当我们要打印比较复杂的日志信息时。比如，要将函数的输出重定向到一个文件中，或者将函数的打印输出处于静默状态，而不需要对函数进行任何修改。或许这个函数为一个库函数，我们没有办法看到其源码。可能，这个函数并没有设计为写入到文件的函数，但是我们还是想将其输出输入到文件中。

这里可以重定向输出流对象的输出。本节中，我们将看到如何使用一种简单并且优雅地方式来完成输出流的重定向。

## How to do it...

我们将实现一个辅助类，其能在构造和析构阶段，帮助我们完成流的重定向，以及对流的定向进行恢复。然后，我们来看其是怎么使用的：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <fstream>
   
   using namespace std;
   ```

2. 我们实现了一个类，其具有一个文件输出流对象和一个指向流内部缓冲区的指针。`cout`作为流对象，其内部具有一个缓冲区，其可以用来进行数据交换，我们可以保存我们之前做过的事情，这样就很方便进行对后续修改的撤销。我们可以在C++手册中查询对其返回类型的解释，也可以使用`decltype`对`cout.rdbuf()`所返回的类型进行查询。这并不是一个很好的体验，在我们的例子中，其就是一个指针类型：

   ```c++
   class redirect_cout_region
   {
       using buftype = decltype(cout.rdbuf());
       
       ofstream ofs;
       buftype buf_backup; 
   ```

3. 类的构造函数接受一个文件名字符串作为输入参数。这个字符串用来初始化文件流成员`ofs`。对其进行初始化后，可以将其输入到`cout`作为一个新的流缓冲区。`rdbuf`在接受一个新缓冲区的同时，会将旧缓冲区以指针的方式进行返回，这样当需要对缓冲区进行恢复时，就可以直接使用了：

   ```c++
   public:
       explicit
       redirect_cout_region (const string &filename)
       : ofs{filename}
   	, buf_backup{cout.rdbuf(ofs.rdbuf())}
       {}
   ```

4. 默认构造函数和其他构造函数做的事情几乎一样。其区别在于，默认构造函数不会打开任何文件。默认构造的文件流会直接替换`cout`的流缓冲，这样会导致`cout`的一些功能失效。其会丢弃一些要打印的东西。这在某些情况下是非常有用的：

   ```c++
       redirect_cout_region()
       : ofs{}
       ,buf_backup{cout.rdbuf(ofs.rdbuf())}
       {}
   ```

5. 析构函数会对重定向进行恢复。当类在运行过程中超出了范围，可以使用原始的`cout`流缓冲区对其进行还原：

   ```c++
       ~redirect_cout_region() {
       	cout.rdbuf(buf_backup);
       }
   };
   ```

6. 让我们模拟一个有很多输出的函数：

   ```c++
   void my_output_heavy_function()
   {
       cout << "some output\n";
       cout << "this function does really heavy work\n";
       cout << "... and lots of it...\n";
       // ...
   }
   ```

7. 主函数中，我们先会进行一次标准打印：

   ```c++
   int main()
   {
   	cout << "Readable from normal stdout\n";
   ```

8. 现在进行重定向，首先使用一个文本文件名对类进行实例化。文件流会使用读取和写入模式作为默认模式，所以其会创建一个文件。所以即便是后续使用`cout`进行打印，其输出将会重定向到这个文件中：

   ```c++
   	{
           redirect_cout_region _ {"output.txt"};
           cout << "Only visible in output.txt\n";
           my_output_heavy_function();
   	}
   ```

9. 离开这段代码后，文件将会关闭，打印输出也会重归标准输出。我们再开启一个代码段，并使用默认构造函数对类进行构造。这样后续的打印信息将无法看到，都会被丢弃：

   ```c++
       {
           redirect_cout_region _;
           cout << "This output will "
                   "completely vanish\n";
       }
   ```

10. 离开这段代码后，我们的标准输出将再度恢复，并且将程序的最后一行打印出来：

   ```c++
   	cout << "Readable from normal stdout again\n";
   }
   ```

11. 编译并运行这个程序，其输出和我们期望的一致。我们只看到了第一行和最后一行输出：

    ```c++
    $ ./log_regions
    Readable from normal stdout
    Readable from normal stdout again
    ```

12. 我们可以将新文件output.txt打开，其内容如下：

    ```c++
    $ cat output.txt
    Only visible in output.txt
    some output
    this function does really heavy work
    ... and lots of it...
    ```

## How it works...

每个流对象都有一个内部缓冲区，这样的缓冲区可以进行交换。当我们有一个流对象`s`时，我们将其缓冲区存入到变量`a`中，并且为流对象换上了一个新的缓冲区`b`，这段代码就可以完成上述的过程:` a = s.rdbuf(b)`。需要恢复的时候只需要执行`s.rdbuf(a)`。

这就如同我们在本节所做的。另一件很酷的事情是，可以将这些`redirect_cout_region`辅助函数放入堆栈中：

```c++
{
    cout << "print to standard output\n";
    
    redirect_cout_region la {"a.txt"};
    cout << "print to a.txt\n";
    
    redirect_cout_region lb {"b.txt"};
    cout << "print to b.txt\n";
}
cout << "print to standard output again\n";
```

这也应该好理解，通常析构的顺序和构造的顺序是相反的。这种模式是将对象的构造和析构进行紧耦合，其也称作为**资源获得即初始化(RAII)**。

这里有一个很重要的点需要注意——`redirect_cout_region`类中成员变量的初始化顺序：

```c++
class redirect_cout_region {
    using buftype = decltype(cout.rdbuf());
    ofstream ofs;
    buftype buf_backup;
public:
    explicit
    redirect_cout_region(const string &filename)
    : ofs{filename},
    buf_backup{cout.rdbuf(ofs.rdbuf())}
    {}
...
```

我们可以看到，成员`buf_backup`的初始化需要依赖成员`ofs`进行。有趣的是，这些成员初始化的顺序，不会按照初始化列表中给定元素的顺序进行初始化。这里初始化的顺序值与成员变量声明的顺序有关！

> Note：
>
> 当一成员变量需要在另一个成员变量之后进行初始化，其需要在类声明的时候以相应的顺序进行声明。初始化列表中的顺序，对于构造函数来说没有任何影响。