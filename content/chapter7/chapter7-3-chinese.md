# 无需构造获取std::string

`std::string`类是一个十分有用的类，因为其对字符串的处理很方便。其有一个缺陷，当我们想要根据一个字符串获取其子字符串时，我们需要传入一个指针和一个长度变量，两个迭代器或一段拷贝的子字符串。我在之前的章节也这样使用过，消除字符串前后的空格的最后，使用的是拷贝的方式获得前后无空格的字符串。

当我们想要传递一个字符串或一个子字符串到一个不支持`std::string`的库中时，需要提供裸指针，这样的用法就回退到C的时代。与子字符串问题一样，裸指针不携带字符串长度信息。这样的话就需要将指针和字符串长度进行捆绑。

另一个十分简单的方式就是使用`std::string_view`。这个类是C++17添加的新特性，并且能提供将字符串指针与其长度捆绑的方法，其体现了数组引用的思想。

当设计函数时，将`std::string`实例作为参数，但在函数中使用了额外的内存来存储这些字符，以确保原始的字符串不被修改，这时就可以使用`std::string_view`，其可移植性很好，与STL无关。可以让其他库来提供一个`string_view`实现，然后将复杂的实现隐藏在背后，并且可以将其用在我们的STL代码中。这样，`string_view`类就显得非常小，非常好用，因为其能在不同的库间都可以用。

`string_view`另一个很酷的特性，就是可以使用非拷贝的方式引用大字符串中的子字符串。本节将使用`string_view`，从而了解其优点和缺点。我们还会看到如何使用字符串代理来去除字符两端的空格，并不对原始字符串进行修改和拷贝。

## How to do it...

本节，将使用`string_view`的一些特性来实现一个函数，我们将会看到有多少种类型可以输入：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <string_view>
   
   using namespace std; 
   ```

2.  将`string_view`作为函数的参数：

   ```c++
   void print(string_view v)
   { 
   ```

3. 对输入字符串做其他事情之前，将移除字符开头和末尾的空格。将不会对字符串进行修改，仅适用字符串代理获取没有空格字符串。`find_first_not_of`函数将会在字符串找到第一个非空格的字符，适用`remove_prefix`，`string_view`将指向第一个非空格的字符。当字符串只有空格，`find_first_not_of`函数会返回`npos`，其为`size_type(-1)`。`size_type`是一个无符号类型，其可以是一个非常大的值。所以，会在字符串代理的长度和`words_begin`中选择较小的那个：

   ```c++
   	const auto words_begin (v.find_first_not_of(" \t\n"));
   	v.remove_prefix(min(words_begin, v.size()));
   ```

4. 我们对尾部的空格做同样的事情。`remove_suffix`将收缩到代理的大小：

   ```c++
   	const auto words_end (v.find_last_not_of(" \t\n"));
   	if (words_end != string_view::npos) {
   		v.remove_suffix(v.size() - words_end - 1);
   	} 
   ```

5. 现在可以打印字符串代理和其长度：

   ```c++
   	cout << "length: " << v.length()
   		 << " [" << v << "]\n";
   }
   ```

6. 主函数中，将使用`print`的函数答应一系列完全不同的参数类型。首先，会通过`argv`传入`char*`类型的变量，运行时其会包含可执行文件的名字。然后，传入一个`string_view`的实例。然后，使用C风格的静态字符串，并使用`""sv`字面字符构造的`string_view`类型。最后，传入一个`std::string`。`print`函数不需要对参数进行修改和拷贝。这样就没有多余的内存分配发生。对于很多大型的字符串，这将会非常有效：

   ```c++
   int main(int argc, char *argv[])
   {
   	print(argv[0]);
   	print({});
   	print("a const char * array");
   	print("an std::string_view literal"sv);
   	print("an std::string instance"s); 
   ```

7. 这里还没对空格移除特性进行测试。这里也给出一个头尾都有空格的字符串：

   ```c++
   	print(" \t\n foobar \n \t "); 
   ```

8. `string_view`另一个非常酷的特性是，其给予的字符串是不包含终止符的。当构造一个字符串，比如"abc"，没有终止符，`print`函数就能很安全的对其进行处理，因为`string_view`携带字符串的长度信息和指向信息：

   ```c++
   	char cstr[] {'a', 'b', 'c'};
   	print(string_view(cstr, sizeof(cstr)));
   }
   ```

9. 编译并运行程序，就会得到如下的输出，所有字符串都能被正确处理。前后有很多空格的字符串都被正确的处理，`abc`字符串没有终止符也能被正确的打印，而没有任何内存溢出：

   ```c++
   $ ./string_view
   length: 17 [./string_view]
   length: 0 []
   length: 20 [a const char * array]
   length: 27 [an std::string_view literal]
   length: 23 [an std::string instance]
   length: 6 [foobar]
   length: 3 [abc]
   ```

## How it works...

我们可以看到，函数可以接受传入一个`string_view`的参数，其看起来与字符串类型没有任何区别。我们实现的`print`，对于传入的字符串不进行任何的拷贝。

对于`print(argv[0])`的调用是非常有趣的，字符串代理会自动的推断字符串的长度，因为需要将其适用于无终止符的字符串。另外，我们不能通过查找终止符的方式来确定`string_view`实例的长度。正因如此，当使用裸指针(`string_view::data()`)的时候就需要格外小心。通常字符串函数都会认为字符串具有终止符，这样就很难出现使用裸指针时出现内存溢出的情况。这里还是使用字符串代理的接口比较好。

除此之外，`std::string`接口阵容已经非常豪华了。

> Note：
>
> 使用`std::string_view`用于解析字符或获取子字符串时，能避免多余的拷贝和内存分配，并且还不失代码的舒适感。不过，对于`std::string_view`将终止符去掉这点，需要特别注意。

