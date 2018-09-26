# 删除词组间连续的空格

我们会经常从输入中读取字符串，这些字符串会包含一些原生格式，需要进行清洗。其中一个例子就是字符串中包含了太多的空格。

本节，我们将实现一个聪明的空格滤波算法，其会删除多于的空格，会给单词间留下一个空格。我们可以将这个算法称为`remove_multi_whitespace`，并且接口与STL很像。

## How to do it...

本节，我们将实现过滤空格的算法，并了解其是如何进行工作的：

1. 包含必要的头文件和声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <string>
   #include <algorithm>
   
   using namespace std;
   ```

2. `remove_multi_whitespace`看起来与STL的风格非常类似。这个算法会移除多余的空格，只保留一个空格。当字符串为`a b`，算法是不会进行任何操作的；当字符串为`a   b`时，算法会返回`a b`。为了完成这个算法，我们使用`std::unqiue`通过对一段区域的迭代，用来查找一对连续的元素。然后，通过谓词函数进行判断，确定两个元素是否相等。如果相等，那么`std::unique`会将其中一个移除。这样，子范围中就不会存在相等的元素了。谓词函数会通过读取到的内容来判断二者是否相等。我们需要给`std::unique`怎么样一个谓词函数呢？其需要判断两个元素是否是连续的空格；如果是，就要移除一个空格。与`std::unique`类似，也需要传入一对`begin/end`迭代器，然后返回的迭代器将返回新范围的末尾迭代器：

   ```c++
   template <typename It>
   It remove_multi_whitespace(It it, It end_it)
   {
       return unique(it, end_it, [](const auto &a, const auto &b) {
       	return isspace(a) && isspace(b);
       });
   }
   ```

3. 万事俱备，就来进行测试，尝试使用算法将不必要的空格进行删除：

   ```c++
   int main()
   {
       string s {"fooo bar \t baz"};
       
       cout << s << '\n';
   ```

4. 对字符串使用过滤算法，去掉多余的空格：

   ```c++
       s.erase(remove_multi_whitespace(begin(s), end(s)), end(s));
       
   	cout << s << '\n';
   }
   ```

5. 编译并运行程序，就会得到如下的输出：

   ```c++
   $ ./remove_consecutive_whitespace
   fooo bar       baz
   fooo bar baz
   ```

## How it works...

整个问题的解决中，我们没有使用循环或者元素间的互相比较。我们只使用谓词函数来完成判断两个给定字符是否是空格的任务。然后，将谓词函数与`std::unique`相结合，所有多余的空格就都消失了。本章中有些算法可能会有些争议，不过这个算法的确算的上短小精悍的典范了。

我们如何在将算法进行组合的呢？我们来看一下`std::unique`可能的实现代码：

```c++
template<typename It, typename P>
It unique(It it, It end, P p)
{
	if (it == end) { return end; }

    It result {it};
	while (++it != end) {
		if (!p(*result, *it) && ++result != it) {
			*result = std::move(*it);
		}
	}
	return ++result;
}
```

其中循环会迭代到范围的最后，当元素满足谓词条件，就会从原始位置上移除一个元素。这个版本的`std::unique`不接受多余的谓词函数，来判断两个相邻的元素是否相等。这样的话，只能将重复的字符去除，比如会将`abbbbbbbbc`变换成`abc`。

那么，我们应该怎么做才能不去除除了空格之外的重复的元素呢？这样，谓词函数不能告诉程序“两个输入字符是相同的”，而是要说“两个输入字符都是空格”。

最后需要注意的是，无论是`std::unique`还是`remove_multi_whitespace`都会从字符串中移除字母元素。根据字符串的语义来移动字符串，并表明新的结尾在哪里。新的尾部到旧的尾部的元素依旧存在，所以我们必须将它们删除：

```c++
s.erase(remove_multi_whitespace(begin(s), end(s)), end(s));
```

和`vector`和`list`一样，`erase`成员函数其会对元素进行擦除和删除。

