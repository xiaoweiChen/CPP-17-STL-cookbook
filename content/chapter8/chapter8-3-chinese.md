# 安全的标识失败——std::optional

当程序与外界的联系只依赖于一些变量时，那么各种失败都可能发生。

也就是，我们写了一个函数，其会返回一个值，但是当函数接口进行变更后，可能就无法获取这个返回值了。我们来看下对一个返回字符串的函数，怎样的接口会容易出现失败的情况：

- 使用引用值作为返回值：`bool get_string(string&);`
- 返回一个可以被设置为nullptr的指针(或智能指针)：`string* get_string();`
- 当函数出错时，直接抛出异常：`string get_string();`

以上的方式有缺点，也有优点。在C++17之后，我们会使用一种新类型来解决这个问题：`std::optional`。可选值的概念来自于纯函数式编程语言(在纯函数式语言中，这个类型为Maybe类型)，并且可以让代码看上去很优雅。

我们可以将`optional`包装到我们的类型中，其可以表示空值或错误值。本节中，我们就会来学习怎么使用这个类型。

## How to do it...

本节，我们将实现一个程序用于从用户输入中读取整型数，然后将这些数字加起来。因为不确定用户会输入什么，所以我们会使用`optional`进行错误处理：

1. 包含必要的头文件，并声明所使用的命名空间。

   ```c++
   #include <iostream>
   #include <optional>
   
   using namespace std; 
   ```

2. 定义一个整型类型，其可能会包含一个值，使用`std::optional`类型来完成这件事。将目标类型包装进`optional`，我们会给其一个附加状态，其表示当前对象中没有值：

   ```c++
   using oint = optional<int>;
   ```

3. 使用包装后的整型类型，我们用其来表示函数返回失败的情况。当从用户输入中获取一个整数时，这个函数可能会失败，因为用户可能输入的就不是我们想要的东西，返回可选整型就能很好的解决这个问题。当成功的读取一个整数，我们会将其放入`optional<int>`的构造函数中。否则，我们将返回一个默认构造的`optional`，其代表没有获取成功：

   ```c++
   oint read_int()
   {
       int i;
       if (cin >> i) { return {i}; }
       return {};
   }
   ```

4. 除了获取整数，我们还能做的更多。那怎么使用两个可选整数进行相加呢？如果两个可选整数中具有相应的整数值，那么使用实际的数值直接相加。存在有空的可选变量时，我们会返回一个空的可选变量。这个函数需要简单的来解释一下：通过隐式转换，将`optional<int>`变量a和b转化成一个布尔表达式(写成!a和!b)，这就能让我们确定可选变量中是否有值。如果其中有值，我们将对其使用指针或是迭代器的方式，对a和b直接解引用：

   ```c++
   oint operator+(oint a, oint b)
   {
       if (!a || !b) { return {}; }
       return {*a + *b};
   }
   ```

5. 重载加法操作，可以直接和一个普通整数进行相加：

   ```c++
   oint operator+(oint a, int b)
   {
       if (!a) { return {}; }
       
       return {*a + b};
   }
   ```

6. 现在来完成主函数部分，我们会让用户输入两个数值：

   ```c++
   int main()
   {
       cout << "Please enter 2 integers.\n> ";
       
       auto a {read_int()};
       auto b {read_int()}; 
   ```

7. 然后，将获取的数值进行相加，并再与10进行相加。这里`a`和`b`为可选整型类变量，`sum`也为可选整型类变量：

   ```c++
   	auto sum (a + b + 10);
   ```

8. 当`a`和/或`b`中不包含一个值时，`sum`就也不包含任何值。可选整型可依然我们不必显式的对`a`和`b`进行检查。当遇到空值的时，我们定义的操作符能很完美的处理这样的情况。这样，我们只需要对结果可选整型变量进行检查即可。如果包含一个值，那就可以安全的对这个值进行访问，并将其进行打印：

   ```c++
   if (sum) {
   	cout << *a << " + " << *b << " + 10 = "
   		<< *sum << '\n';
   ```

9. 当用户输入了非数字内容，我们将会输出错误信息：

   ```c++
       } else {
           cout << "sorry, the input was "
           		"something else than 2 numbers.\n";
       }
   }
   ```

10. 完成了！编译并运行程序，我们将会得到如下输出：

    ```c++
    $ ./optional
    Please enter 2 integers.
    > 1 2
    1 + 2 + 10 = 13
    ```

11. 当输入中包含非数字元素，我们将会得到如下输出：

    ```c++
    $ ./optional
    Please enter 2 integers.
    > 2 z
    sorry, the input was something else than 2 numbers.
    ```

## How it works...

`optional`非常简单易用。其可以帮助我们对错误的情况进行处理，当我们所需要的类型为T时，可以将其特化`std::optional<T>`版本类型进行封装。

当需要从一些地方获取一些值时，我们可以用其来检查我们是否成功的获取了对应的数值。`bool optional::has_value()`可以帮助我们完成这件事。当其包含值时，其会返回true，我们就能直接对数值进行访问，对可选类型的值访问也可以通过函数`T& optional::value()`进行。

例子中，使用`  if (x) {...}`和`*x`来替代`if (x.has_value()) {...}`和`x.value() `。`std::optonal`类型可以隐式的转换成`bool`类型，并且使用解引用操作符的方式和普通指针差不多。

另一个方便辅助操作符就是对`optional`的`operator->`操作符进行重载。当有一个结构体`  struct Foo { int a; string b; }`类型，并且我们想要通过一个`optional<Foo>`来访问其成员变量x，那么就可以写成`x->a`或`x->b`。当然，需要对x和b进行检查，确定其是否有值。

当可选变量中没有值时，我们还要对其进行访问，其就会抛出一个`std::logic_error`异常。这样，就可以对大量的可选值在不进行检查的情况下进行使用。`try-catch`块的代码如下：

```c++
cout << "Please enter 3 numbers:\n";

try {
	cout << "Sum: "
		<< (*read_int() + *read_int() + *read_int())
		<< '\n';
} catch (const std::bad_optional_access &) {
	cout << "Unfortunately you did not enter 3 numbers\n";
}
```

`std::optional`具有一个有趣的`optional::value_or`操作。当我们想要在失败的时候，可选变量包含一个默认值进行返回时，这个操作就很有用了。`x = optional_var.value_or(123) `就能将123作为可选变量失败时的默认数值。