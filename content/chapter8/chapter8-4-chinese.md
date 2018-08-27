# 对元组使用函数

C++11中，STL添加了`std::tuple`，这种类型可以用来将多个不同类型的值捆绑在一起。元组这种类型已经存在与很多编程语言中，本书的一些章节已经在使用这种类型，这种类型的用途很广泛。

不过，我们有时会将一些值捆绑在一个元组中，然后我们需要调用函数来获取其中每一个元素。对于元素的解包的代码看起来非常的冗长(并且易于出错)。其冗长的方式类似这样：`func(get<0>(tup), get<1>(tup), get<2>(tup), ...);`。

本节中，你将了解如何使用一种优雅地方式对元组进行打包和解包。调用函数时，你无需对元组特别地了解。

## How to do it...

我们将实现一个程序，其能对元组值进行打包和解包。然后，我们将看到在不了解元组中元素的情况下，如何使用元组：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <iomanip>
   #include <tuple>
   #include <functional>
   #include <string>
   #include <list>
   
   using namespace std;
   ```

2. 首先定义一个函数，这个函数能接受多个参数，其描述的是一个学生，并将学生的相关信息进行打印。其和C风格的函数看起来差不多：

   ```c++
   static void print_student(size_t id, const string &name, double gpa)
   {
       cout << "Student " << quoted(name)
           << ", ID: " << id
           << ", GPA: " << gpa << '\n';
   }
   ```

3. 主函数中，将对一种元组类型进行别名，然后将具体学生的信息填入到这种类型的实例中：

   ```c++
   int main()
   {
       using student = tuple<size_t, string, double>;
       student john {123, "John Doe"s, 3.7};
   ```

4. 为了打印这种类型的实例，我们将会对元组中的元素进行分解，然后调用`print_student`函数将这些值分别进行打印：

   ```c++
   	{
           const auto &[id, name, gpa] = john;
           print_student(id, name, gpa);
       }
       cout << "-----\n";
   ```

5. 然后，我们来创建一个以元组为基础类型的多个学生：

   ```c++
       auto arguments_for_later = {
           make_tuple(234, "John Doe"s, 3.7),
           make_tuple(345, "Billy Foo"s, 4.0),
           make_tuple(456, "Cathy Bar"s, 3.5),
       };
   ```

6. 这里，我们依旧可以通过对元素进行分解，然后对其进行打印。当要写这样的代码时，我们需要在函数接口变化时，对代码进行重构：

   ```c++
       for (const auto &[id, name, gpa] : arguments_for_later) {
      		print_student(id, name, gpa);
       }
       cout << "-----\n";
   ```

7. 当然可以做的更好，我们无需知道`print_student`的参数的个数，或学生元组中元素的个数，我们使用`std::apply`对直接将元组应用于函数。这个函数能够接受一个函数指针或一个函数对象和一个元组，然后会将元组进行解包，然后与函数参数进行对应，并传入函数：

   ```c++
   	apply(print_student, john);
   	cout << "-----\n";
   ```

8. 循环中可以这样用：

   ```c++
       for (const auto &args : arguments_for_later) {
       	apply(print_student, args);
       }
       cout << "-----\n";
   }
   ```

9. 编译并运行程序，我们就能得到如下的输出：

   ```c++
   $ ./apply_functions_on_tuples
   Student "John Doe", ID: 123, GPA: 3.7
   -----
   Student "John Doe", ID: 234, GPA: 3.7
   Student "Billy Foo", ID: 345, GPA: 4
   Student "Cathy Bar", ID: 456, GPA: 3.5
   -----
   Student "John Doe", ID: 123, GPA: 3.7
   -----
   Student "John Doe", ID: 234, GPA: 3.7
   Student "Billy Foo", ID: 345, GPA: 4
   Student "Cathy Bar", ID: 456, GPA: 3.5
   -----
   ```

## How it works...

`std::apply`是一个编译时辅助函数，可以帮助我们处理不确定的类型参数。

试想，我们有一个元组`t`，其有元素`(123, "abc"s, 456.0)`。那么这个元组的类型为` tuple<int, string, double>`。另外，有一个函数`f`的签名为`int f(int, string, double)`(参数类型也可以为引用)。

然后，我们就可以这样调用函数`x = apply(f, t)`，其和`x = f(123, "abc"s, 456.0)`等价。`apply`方法还是会返回`f`的返回值。

