# 过滤用户的重复输入，并以字母序将重复信息打印出——std::set

`std::set`是一个奇怪的容器：工作原理和`std::map`很像，不过`std::set`将键作为值，没有键值对。所以没做办法与其他类型的数据进行映射。表面上看，`std::set`因为没有太多的例子，导致很多开发者几乎不知道有这样的容器。想要使用类似`std::set`的功能时，只有自己去实现一遍。

本节展示如何使用`std::set`收集很多不同的元素，过滤这些元素，最后只输出一个元素。

## How to do it...

从标准输入流中读取单词，所有不重复的单词将放到一个`std::set`实例中。之后，枚举出所有输入流中不重复的单词。

1. 包含必要的头文件。

   ```c++
   #include <iostream>
   #include <set>
   #include <string>
   #include <iterator>
   ```

2. 为了分析我们的输入，会使用到`std`命名空间：

   ```c++
   using namespace std;
   ```

3. 现在来实现主函数，先来实例化一个`std::set`。

   ```c++
   int main()
   {
   	set<string> s;
   ```

4. 下一件事情就是获取用户的输入。我们只从标准输入中读取，这样我们就要用到`istream_iterator`。

   ```c++
       istream_iterator<string> it {cin};
       istream_iterator<string> end;
   ```

5. 这样就得到了一对`begin`和`end`迭代器，可以用来表示用户的输入，我们可以使用`std::inserter`来填满`set`实例。

   ```c++
   	copy(it, end, inserter(s, s.end()));
   ```

6. 这样就完成了填充。为了看到从标准输入获得的不重复单词，我们可以打印当前的`set`实例。

   ```c++
   	for (const auto word : s) {
       	cout << word << ", ";
       }
       cout << '\n';
   }
   ```

7. 最后，让我们编译并运行这个程序。从之前的输入中，重复的单词都会去除，获得不重复的单词，然后以字母序排序输出。

   ```
   $ echo "a a a b c foo bar foobar foo bar bar" | ./program
   a, b, bar, c, foo, foobar,
   ```

## How it works...

程序中有两个有趣的部分。第一个是使用了`std::istream_iterator`来访问用户输入，另一个是将`std::set`实例使用`std::inserter`用包装后，在使用`std::copy`填充。这看起来像是变魔术一样，只需要一行代码，我们就能完成使用输入流填充实例，去除重复的单词和以字母序进行排序。

**std::istream_iterator**

这个例子的有趣之处在于一次性可以处理流中大量相同类型的数据：我们对整个输入进行逐字的分析，并以`std::string`实例的方式插入`set`。

`std::istream_iterator`只传入了一个模板参数。也就我们输入数据的类型。我们选择`std::string`是因为我们假设是文本输入，不过这里也可以是`float`型的输入。基本上任何类型都可以使用`cin >> var;`完成。构造函数接受一个`istream`实例。标准输入使用全局输入流`std::cin`表示，例子中其为`istream`的参数。

```c++
istream_iterator<string> it {cin};
```

输入流迭代器`it`就已经实例化完毕了，其可以做两件事情：当对其解引用(`*it`)时，会得到当前输入的符号。我们通过输入迭代器构造`std::string`实例，每个字符串容器中都包含一个单词；当进行自加`++it`时，其会跳转到下一个单词，然后再解引用访问下一个单词。

不过，每次自加后的解引用时都须谨慎。当标准输入为空，迭代器就不能再解引用。另外，我们需要终止使用解引用获取单词的循环。终止的条件就是通过和`end`迭代器进行比较，知道何时迭代器无法解引用。如果`it==end`成立，那么说明输入流已经读取完毕。

我们在创建`it`的同时，也创建了一个`std::istream_iterator`的`end`迭代器。其目的是于`it`进行比较，在每次迭代中作为中止条件。

当`std::cin`结束时，`it`迭代器将会与`end`进行比较，并返回true。

**std::inserter**

调用`std::copy`时，我们使用`it`和`end`作为输入迭代器。第三个参数必须是一个输出迭代器。因此，不能使用`s.begin()`或`s.end()`。一个空的`set`中，这二者是一致的，所以不能对`set`的迭代器进行解引用(无论是读取或赋值)。

这就使`std::inserter`有了用武之地。其为一个函数，返回一个`std::insert_iterator`，返回值的行为类似一个迭代器，不过会完成普通迭代器无法完成的事。当对其使用加法时，其不会做任何事。当我们对其解引用，并赋值给它时，它会连接相关容器，并且将赋值作为一个新元素插入容器中。

当通过`std::inserter`实例化`std::insert_iterator`时，我们需要两个参数：

```c++
auto insert_it = inserter(s, s.end());
```

其中s是我们的`set`，`s.end()`是指向新元素插入点的迭代器。对于一个空`set`来说，从哪里开始和从哪里结束一样重要。当使用其他数据结构时，比如`vector`和`list`，第二个参数对于定义插入新项的位置来说至关重要。

**将二者结合**

最后，所有的工作都在`std::copy`的调用中完成：

```c++
copy(input_iterator_begin, input_iterator_end, insert_iterator);
```

这个调用从`std::cin`中获取输入迭代器，并将其推入`std::set`中。之后，其会让迭代器自增，并且确定输入迭代器是否达到末尾。如果不是，那么可以继续从标准输入中获取单词。

重复的单词会自动去除。当`set`已经拥有了一个单词，再重复将这个单词添加入`set`时，不会产生任何效果。与`std::multiset`的表现不同，`std::multiset`会接受重复项。