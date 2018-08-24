# 简单打印不同格式的数字

之前的章节中，我们已经了解如何打印出带格式的输出，同时也意识到了两点：

- 输入输出控制符是有粘性的，所以当我们要临时使用的时候，需要在用完之后进行还原。
- 其控制符和较少的要打印的对象相比，会显得很冗长。

这些原因导致一些开发者使用C++的时候，还是依旧使用`printf`进行打印输出。

本节，我们将来看一下如何不用太多代码就能进行很好的类型打印。

## How to do it...

我们会先来实现一个类`format_guard`，其会自动的将打印格式进行恢复。另外，我们添加了一个包装类型，其可以包含任意值，当对其进行打印时，其能使用相应的格式进行输出，而无需添加冗长的控制符：

1. 包含必要的头文件，并声明所使用的命名空间。

   ```c++
   #include <iostream>
   #include <iomanip>
   
   using namespace std; 
   ```

2. 辅助类在调用`format_guard`时，其会对输出流的格式进行清理。其构造函数保存了格式符，也就是在这里对`std::cout`进行设置。析构函数会将这些状态进行去除，这样就不会后续的打印有所影响：

   ```c++
   class format_guard {
   	decltype(cout.flags()) f {cout.flags()};
   public:
   	~format_guard() { cout.flags(f); }
   };
   ```

3. 定义另一个辅助类`scientific_type`。因为其是一个模板类，所以其能拥有任意类型的成员变量。这个类没有其他任何作用：

   ```c++
   template <typename T>
   struct scientific_type {
   	T value;
       
   	explicit scientific_type(T val) : value{val} {}
   };
   ```

4. 封装成`scientific_type`之后，可以对任意类型进行自定义格式设置，当对`operator>>`进行重载后，输出流就会在执行时，运行完全不同的代码。这样就能在使用科学计数法表示浮点数时，以大写的格式，并且其为正数时，数字前添加'+'号。我们也会在跳出函数时，使用`format_guard`类对打印格式进行清理：

   ```c++
   template <typename T>
   ostream& operator<<(ostream &os, const scientific_type<T> &w) {
       format_guard _;
       os << scientific << uppercase << showpos;
       return os << w.value;
   }
   ```

5. 主函数中，我们将使用到`format_guard`类。我们会创建一段新的代码段，首先对类进行实例化，并且对`std::cout`进行输出控制符的设置：

   ```c++
   int main()
   {
       {
           format_guard _;
           cout << hex << scientific << showbase << uppercase;
           
           cout << "Numbers with special formatting:\n";
           cout << 0x123abc << '\n';
           cout << 0.123456789 << '\n';
       }
   ```

6. 使用控制符对这些数字进行打印后，跳出这个代码段。这时`format_guard`的析构函数会将格式进行清理。为了对清理结果进行测试，会再次打印相同的数字。其将会输出不同的结果：

   ```c++
   	cout << "Same numbers, but normal formatting again:\n";
   	cout << 0x123abc << '\n';
   	cout << 0.123456789 << '\n';
   ```

7. 现在使用`scientific_type`，将三个浮点数打印在同一行。我们将第二个数包装成`scientific_type`类型。这样其就能按照我们指定的风格进行打印，不过在之前和之后的输出都是以默认的格式进行。与此同时，我们也避免了冗长的格式设置代码：

   ```c++
       cout << "Mixed formatting: "
           << 123.0 << " "
           << scientific_type{123.0} << " "
           << 123.456 << '\n';
   }
   ```

8. 编译并运行程序，我们就会得到如下的输出。前两行按照我们的设定进行打印。接下来的两行则是以默认的方式进行打印。这样就证明了我们的`format_guard`类工作的很好。最后三个数在一行上，也是和我们的期望一致。只有中间的数字是`scientific_type`类型的，前后两个都是默认类型：

   ```c++
   $ ./pretty_print_on_the_fly
   Numbers with special formatting:
   0X123ABC
   1.234568E-01
   Same numbers, but normal formatting again:
   1194684
   0.123457
   Mixed formatting: 123 +1.230000E+02 123.456
   ```

   