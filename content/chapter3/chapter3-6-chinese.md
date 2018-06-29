# 使用哨兵终止迭代

对于STL算法和基于范围的for循环来说，都会假设迭代的位置是提前知道的。在有些情况下，并不是这样，我们在迭代器到达末尾之前，我们是很难确定结束的位置在哪里。

这里使用C风格的字符串来举例，我们在编译时无法知道字符串的长度，只能在运行时使用某种方法进行判断。字符串遍历的代码如下所示：

```c
for (const char *c_ponter = some_c_string; *c_pointer != '\0'; ++c_pointer) {
    const char c = *c_pointer;
    // do something with c
}
```

对于基于范围的for循环来说，我们可以将这段字符串包装进一个`std::string`实例中，`std::string`提供`begin()`和`end()`函数：

```c++
for (char c : std::string(some_c_string)) { /* do something with c */ }
```

不过，`std::string`在构造的时候也需要对整个字符串进行遍历。C++17中加入了`std::string_view`，但在构造的时候也会对字符串进行一次遍历。对于比较短的字符串来说这是没有必要的，不过对于其他类型来说就很有必要。`std::istream_iterator`可以用来从`std::cin`捕获输入，当用户持续输入的时候，其`end`迭代器并不能指向输入字符串真实的末尾。

C++17添加了一项新的特性，其不需要`begin`迭代器和`end`迭代器是同一类型的迭代器。本节我们看看，这种小修改的大用途。

## How to do it...

本节，我们将在范围类中构造一个迭代器，其就不需要知道字符串的长度，也就不用提前找到字符串结束的位置。

1. 包含必要的头文件。

   ```c++
   #include <iostream> 
   ```

2. 迭代器哨兵是本节的核心内容。奇怪的是，它的定义完全是空的。

   ```c++
   class cstring_iterator_sentinel {};
   ```

3. 我们先来实现迭代器。其包含一个字符串指针，指针指向的容器就是我们要迭代的：

   ```c++
   class cstring_iterator {
   	const char *s {nullptr};
   ```

4. 构造函数只是初始化内部字符串指针，对应的字符串是外部输入。显式声明构造函数是为了避免字符串隐式转换为字符串迭代器:

   ```c++
   public:
       explicit cstring_iterator(const char *str)
       	: s{str}
       {}
   ```

5. 当对迭代器进行解引用，其就会返回对应位置上的字符：

   ```c++
   	char operator*() const { return *s; }
   ```

6. 累加迭代器只增加迭代器指向字符串的位置：

   ```c++
       cstring_iterator& operator++() {
           ++s;
           return *this;
       }
   ```

7. 这一步是最有趣的。我们为了比较，实现了`!=`操作符。不过，这次我们不会去实现迭代器的比较操作，这次迭代器要和哨兵进行比较。当我们比较两个迭代器时，在当他们指向的位置相同时，我们可以认为对应范围已经完成遍历。通过和空哨兵对象比较，当迭代器指向的字符为`\0`字符时，我们可以认为到达了字符串的末尾。

   ```c++
       bool operator!=(const cstring_iterator_sentinel) const {
       	return s != nullptr && *s != '\0';
       }
   };
   ```

8. 为了使用基于范围的`for`循环，我们需要一个范围类，用来指定`begin`和`end`迭代器：

   ```c++ 
   class cstring_range {
   	const char *s {nullptr};
   ```

9. 实例化时用户只需要提供需要迭代的字符串：

   ```c++
   public:
       cstring_range(const char *str)
       	: s{str}
       {}
   ```

10. `begin()`函数将返回一个`cstring_iterator`迭代器，其指向了字符串的起始位置。`end()`函数会返回一个哨兵类型。需要注意的是，如果不使用哨兵类型，这里将返回一个迭代器，这个迭代器要指向字符串的末尾，但是我们无法预知字符串的末尾在哪里。

   ```c++
       cstring_iterator begin() const {
      		return cstring_iterator{s};
       }
       cstring_iterator_sentinel end() const {
       	return {};
       }
   };
   ```

11. 类型定义完，我们就来使用它们。例子中字符串是用户输入，我们无法预知其长度。为了让使用者给我们一些输入，我们的例子会判断是否有输入参数。

    ```c++
    int main(int argc, char *argv[])
    {
        if (argc < 2) {
            std::cout << "Please provide one parameter.\n";
            return 1;
        }
    ```

12. 当程序运行起来时，我们就知道`argv[1]`中包含的是使用者的字符串。

    ```c++
        for (char c : cstring_range(argv[1])) {
        	std::cout << c;
        }
        std::cout << '\n';
    } 
    ```

13. 编译运行程序，就能得到如下的输出：

    ```c++
    $ ./main "abcdef"
    abcdef
    ```

循环会将所有的字符打印出来。这是一个很小的例子，只是为了展示如何使用哨兵确定迭代的范围。当在无法获得`end`迭代器的位置时，这是一种很有用的方法。当能够获得`end`迭代器时，就不需要使用哨兵了。