# 将`void*`替换为更为安全的std::any

有时我们会需要将一个变量保存在一个未知类型中。对于这样的变量，我们通常会对其进行检查，以确保其是否包含一些信息，如果是包括，那我们将会去判别所包含的内容。以上的所有操作，都需要在一个类型安全的方法中进行。

以前，我们会将可变对象存与`void*`指针当中。void类型的指针无法告诉我们其所指向的对象类型，所以我们需要将其进行手动转换成我们期望的类型。这样的代码看起来很诡异，并且不安全。

C++17在STL中添加了一个新的类型——`std::any`。其设计就是用来持有任意类型的变量，并且能提供类型的安全检查和安全访问。

本节中，我们将会来感受一下这种工具类型。

## How to do it...

我们将实现一个函数，这个函数能够打印所有东西。其就使用`std::any`作为参数：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <iomanip>
   #include <list>
   #include <any>
   #include <iterator>
   
   using namespace std;
   ```

2. 为了减少后续代码中尖括号中的类型数量，我们对`list<int>`进行了别名处理： 

   ```c++
   using int_list = list<int>;
   ```

3. 让我们实现一个可以打印任何东西的函数。其确定能打印任意类型，并以`std::any`作为其参数：

   ```c++
   void print_anything(const std::any &a)
   {
   ```

4. 首先，要做的事就是对传入的参数进行检查，确定参数中是否包含任何东西，还是只是一个空实例。如果为空，那就没有必要再进行接下来的打印了：

   ```c++
   	if (!a.has_value()) {
   		cout << "Nothing.\n";
   ```

5. 当非空时，就要需要对其进行类型比较，直至匹配到对应类型。这里第一个类型为`string`，当传入的参数是一个`string`，我们可以使用`std::any_cast`将`a`转化成一个`string`类型的引用，然后对其进行打印。我们将双引号当做打印字符串的修饰：

   ```c++
       } else if (a.type() == typeid(string)) {
           cout << "It's a string: "
           	<< quoted(any_cast<const string&>(a)) << '\n';
   ```

6. 当其不是`string`类型时，其也可能是一个`int`类型。当与之匹配是使用`any_cast<int>`将`a`转换成`int`型数值：

   ```c++
       } else if (a.type() == typeid(int)) {
       	cout << "It's an integer: "
       		<< any_cast<int>(a) << '\n';
   ```

7. `std::any`并不只对`string`和`int`有效。我们将`map`或`list`，或是更加复杂的数据结构放入一个`any`变量中。让我们输入一个整数列表看看，按照我们的预期，函数也将会打印出相应的列表：

   ```c++
       } else if (a.type() == typeid(int_list)) {
           const auto &l (any_cast<const int_list&>(a));
          
           cout << "It's a list: ";
           copy(begin(l), end(l),
           	ostream_iterator<int>{cout, ", "});
           cout << '\n'; 
   ```

8. 如果没有类型能与之匹配，那就不会进行猜测了。我们会放弃对类型进行匹配，然后告诉使用者，我们对输入毫无办法：

   ```c++
       } else {
       	cout << "Can't handle this item.\n";
       }
   }
   ```

9. 主函数中，我们能够对调用函数传入任何类型的值。我们可以使用大括号对来构建一个空的`any`变量，或是直接输入字符串“abc”，或是一个整数。因为`std::any`可以由任何类型隐式转换而成，这里并没有语法上的开销。我们也可以直接构造一个列表，然后丢入函数中：

   ```c++
   int main()
   {
       print_anything({});
       print_anything("abc"s);
       print_anything(123);
       print_anything(int_list{1, 2, 3});
   ```

10. 当我们想要传入的参数比较大，那么拷贝到`any`变量中就会花费很长的时间，这是可以使用立即构造的方式。`in_place_type_t<int_list>{}`表示一个空的对象，对于`any`来说其就能够知道应该如何去构建对象了。第二个参数为`{1,2,3}`其为一个初始化列表，其会用来初始化`int_list`对象，然后被转换成`any`变量。这样，我们就避免了不必要的拷贝和移动：

    ```c++
    	print_anything(any(in_place_type_t<int_list>{}, {1, 2, 3}));
    }
    ```

11. 编译并运行程序，我们将得到如下的输入出：

    ```c++
    $ ./any
    Nothing.
    It's a string: "abc"
    It's an integer: 123
    It's a list: 1, 2, 3,
    It's a list: 1, 2, 3,
    ```

## How it works...

`std::any`类型与`std::optional`类型很类似——具有一个`has_value()`成员函数，能告诉我们其是否携带一个值。不过这里，我们还需要对字面的数据进行保存，所以`any`要比`optional`类型复杂的多。

访问any变量的内容前，我们需要知道其所承载的类型，然后将`any`变量转换成那种类型。

这里，使用的比较方式为`x.type == typeid(T)`。如果比较结果匹配，那么就使用`any_cast`对其内容进行转换。

需要注意的是`any_cast<T>(x)`将会返回`x`中`T`值的副本。如果想要避免对复杂对象不必要的拷贝，那就需要使用`any_cast<T&>(x)`。本节的代码中，我们使用引用的方式来获取`string`和`list<int>`对象的值。

> Note：
>
> 如果`any`变量转换成为一种错误的类型，其将会抛出`std::bad_any_cast`异常。





