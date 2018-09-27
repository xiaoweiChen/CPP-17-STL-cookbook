# 通过集成std::char_traits创建自定义字符串类

我们知道`std::string`非常好用。不过，对于一些朋友来说他们需要对自己定义的字符串类型进行处理。

使用它们自己的字符串类型显然不是一个好主意，因为对于字符串的安全处理是很困难的。幸运的是，`std::string`是`std::basic_string`类型的一个特化版本。这个类中包含了所有复杂的内存处理，不过其对字符串的拷贝和比较没有添加任何条件。所以我们可以基于`basic_string`，将其所需要包含的自定义类作为一个模板参数传入。

本节中，我们将来看下如何传入自定义类型。然后，在不实现任何东西的情况下，如何对自定义字符串进行创建。

## How to do it...

我们将实现两个自定义字符串类：`lc_string`和`ci_string`。第一个类将通过输入创建一个全是小写字母的字符串。另一个字符串类型不会对输入进行任何变化，不过其会对字符串进行大小写不敏感的比较：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <algorithm>
   #include <string>
   
   using namespace std;
   ```

2. 然后要对`std::tolower`函数进行实现，其已经定义在头文件`<cctype>`中。其函数也是现成的，不过其不是`constexpr`类型。C++17中一些`string`函数可以声明成`constexpr`类型，但是还要使用自定义的类型。所以对于输入字符串，只将大写字母转换为小写，而其他字符则不进行修改：

   ```c++
   static constexpr char tolow(char c) {
       switch (c) {
       case 'A'...'Z': return c - 'A' + 'a'; // 读者自行将case展开
       default: 	    return c;
       }
   }
   ```

3.  `std::basic_string`类可以接受三个模板参数：字符类型、字符特化类和分配器类型。本节中我们只会修改字符特化类，因为其定义了字符串的行为。为了重新实现与普通字符串不同的部分，我们会以`public`方式继承标准字符特化类：

   ```c++
   class lc_traits : public char_traits<char> {
   public:
   ```

4. 我们类能接受输入字符串，并将其转化成小写字母。这里有一个函数，其是字符级别的，所以我们可以对其使用`tolow`函数。我们的这个函数为`constexpr`：

   ```c++
   	static constexpr
   	void assign(char_type& r, const char_type& a ) {
   		r = tolow(a);
   	}
   ```

5. 另一个函数将整个字符串拷贝到我们的缓冲区内。使用`std::transform`将所有字符从源字符串中拷贝到内部的目标字符串中，同时将每个字符与其小写版本进行映射：

   ```c++
       static char_type* copy(char_type* dest,
      						 const char_type* src,
       					 size_t count) {
       	transform(src, src + count, dest, tolow);
       	return dest;
       }
   };
   ```

6. 上面的特化类可以帮助我们创建一个字符串类，其能有效的将字符串转换成小写。接下来我们在实现一个类，其不会对原始字符串进行修改，但是其能对字符串做大小写敏感的比较。其继承于标准字符串特征类，这次将对一些函数进行重新实现：

   ```c++
   class ci_traits : public char_traits<char> {
   public:
   ```

7. `eq`函数会告诉我们两个字符是否相等。我们也会实现一个这样的函数，但是我们只实现小写字母的版本。这样'A'与'a'就是相等的：

   ```c++
       static constexpr bool eq(char_type a, char_type b) {
       	return tolow(a) == tolow(b);
       }
   ```

8. `lt`函数会告诉我们两个字符在字母表中的大小情况。这里使用了逻辑操作符，并继续对两个字符使用转换成小写的函数：

   ```c++
       static constexpr bool lt(char_type a, char_type b) {
       	return tolow(a) < tolow(b);
       }	
   ```

9. 最后两个函数都是字符级别的函数，接下来两个函数都为字符串级别的函数。`compare`函数与`strncmp`函数差不多。当两个字符串的长度`count`相等，那么就返回0。如果不相等，会返回一个负数或一个正数，返回值就代表了其中哪一个在字母表中更小。并计算两个字符串中所有字符之间的距离，当然这些都是在小写情况下进行的操作。C++14后，这个函数可以声明成`constexpr`类型：

   ```c++
    	static constexpr int compare(const char_type* s1,
       						   const char_type* s2,
       						   size_t count) {
           for (; count; ++s1, ++s2, --count) {
               const char_type diff (tolow(*s1) - tolow(*s2));
               if (diff < 0) { return -1; }
               else if (diff > 0) { return +1; }
           }
       	return 0;
       }
   ```

10. 我们所需要实现的最后一个函数就是大小写不敏感的`find`函数。对于给定输入字符串`p`，其长度为`count`，我们会对某个字符`ch`的位置进行查找。然后，其会返回一个指向第一个匹配字符位置的指针，如果没有找到则返回`nullptr`。这个函数比较过程中我们需要使用`tolow`函数将字符串转换成小写，以匹配大小写不敏感的查找。不幸的是，我们不能使用`std::find_if`来做这件事，因为其是非`constexpr`函数，所以我们需要自己去写一个循环：

    ```c++
        static constexpr
        const char_type* find(const char_type* p,
                              size_t count,
                              const char_type& ch) {
        const char_type find_c {tolow(ch)};
        
        for (; count != 0; --count, ++p) {
        	if (find_c == tolow(*p)) { return p; }
        }
        	return nullptr;
        }
    };
    ```

11. OK，所有自定义类都完成了。这里我们可以定义两种新字符串类的类型。`lc_string`代表小写字符串，`ci_string`代表大小写不敏感字符串。这两种类型与`std::string`都有所不同：

    ```c++
    using lc_string = basic_string<char, lc_traits>;
    using ci_string = basic_string<char, ci_traits>;
    ```

12. 为了能让输出流接受新类，我们需要对输出流操作符进行重载：

    ```c++
    ostream& operator<<(ostream& os, const lc_string& str) {
    	return os.write(str.data(), str.size());
    }
    
    ostream& operator<<(ostream& os, const ci_string& str) {
    	return os.write(str.data(), str.size());
    }
    ```

13. 现在我们来对主函数进行编写。先让我们创建一个普通字符串、小写字符串和大小写不敏感字符串的实例，然后直接将其进行打印。其在终端上看起来都很正常，不过小写字符串将所有字符转换成了小写：

    ```c++
    int main()
    {
        cout << " string: "
            << string{"Foo Bar Baz"} << '\n'
            << "lc_string: "
            << lc_string{"Foo Bar Baz"} << '\n'
            << "ci_string: "
            << ci_string{"Foo Bar Baz"} << '\n';
    ```

14. 为了测试大小写不敏感字符串，可以实例化两个字符串，这两个字符串只有在大小写方面有所不同。当我们将这两个字符串进行比较时，其应该是相等的：

    ```c++
    	ci_string user_input {"MaGiC PaSsWoRd!"};
    	ci_string password {"magic password!"};
    ```

15. 之后，对其进行比较，然后将匹配的结果进行打印：

    ```c++
        if (user_input == password) {
            cout << "Passwords match: \"" << user_input
            	 << "\" == \"" << password << "\"\n";
        }
    }
    ```

16. 编译并运行程序，其输出和我们期望的相符。开始的三行并未对输入进行修改，除了`lc_string`将所有字符转换成了小写。最后的比较，在大小写不敏感的前提下，也是相等的：

    ```c++
    $ ./custom_string
    string: Foo Bar Baz
    lc_string: foo bar baz
    ci_string: Foo Bar Baz
    Passwords match: "MaGiC PaSsWoRd!" == "magic password!"
    ```

## How it works...

我们完成的所有子类和函数实现，在新手看来十分的不可思议。这些函数签名都来自于哪里？为什么我们为了函数签名，就要对相关功能性的函数进行重新实现呢？

首先，来看一下`std::string`的类声明：

```c++
template <
    class CharT,
    class Traits = std::char_traits<CharT>,
    class Allocator = std::allocator<CharT>
    >
class basic_string;
```

可以看出`std::string`其实就是一个`std::basic_string<char>` 类，并且其可以扩展为`std::basic_string<char, std::char_traits<char>, std::allocator<char>> `。OK，这是一个非常长的类型描述，不过其意义何在呢？这就表示字符串可以不限于有符号类型`char`，也可以是其他类型。其对于字符串类型都是有效的，这样就不限于处理ASCII字符集。当然，这不是我们的关注点。

` char_traits<char>`类包含`basic_string`所需要的算法。` char_traits<char>`可以进行字符串间的比较、查找和拷贝。

`allocator<char>`类也是一个特化类，不过其运行时给字符串进行空间的分配和回收。这对于现在的我们来说并不重要，我们只使用其默认的方式就好。

当我们想要一个不同的字符串类型是，可以尝试对`basic_string`和`char_traits`类中提供的方法进行复用。我们实现了两个`char_traits`子类：`case_insentitive`和`lower_caser`类。我们可以将这两个字符串类替换标准`char_traits`类型。

> Note：
>
> 为了探寻`basic_string`适配的可能性，我们需要查询C++ STL文档中关于`std::char_traits`的章节，然后去了解还有那些函数需要重新实现。