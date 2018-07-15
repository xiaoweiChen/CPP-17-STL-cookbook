# 实现词频计数器——std::map

`std::map`在收集和统计数据方面非常有用，通过建立键值关系，将可修改的对象映射到对应键上，可以很容易的实现一个词频计数器。

## How to do it...

本节中，我们将从标准输入中获取用户的输入，或是从记录一部小说的文本文件。我们会去标记输入单词，并统计一共有多少个单词。

1. 包含必要的头文件。

   ```c++
   #include <iostream>
   #include <map>
   #include <vector>
   #include <algorithm>
   #include <iomanip>
   ```

2. 声明所使用的命名空间。

   ```c++
   using namespace std;
   ```

3. 我们将使用一个辅助函数，对输入中的符号进行处理。

   ```c++
   string filter_punctuation(const string &s)
   {
       const char *forbidden {".,:; "};
       const auto idx_start (s.find_first_not_of(forbidden));
       const auto idx_end (s.find_last_not_of(forbidden));
       return s.substr(idx_start, idx_end - idx_start + 1);
   }
   ```

4. 现在，我们来实现真正要工作的部分。使用`map`表对输入的每个单词进行统计。另外，使用一个变量来保存目前为止看到的最长单词的长度。程序的最后，我们将打印这个`map`表。

   ```c++
   int main()
   {
       map<string, size_t> words;
       int max_word_len {0};
   ```

5. 将标准输入导入`std::string`变量中，标准输入由空格隔开。通过如下方法获取输入单词。

   ```c++
       string s;
       while (cin >> s) {
   ```

6. 我们获得的单词可能包含标点符号，因为这些符号可能紧跟在单词后面。使用辅助函数将标点符号去除。

   ```c++
   		auto filtered (filter_punctuation(s));
   ```

7. 如果当前处理的单词是目前处理最长的单词，我们会更新`max_word_len`变量。

   ```c++
   		max_word_len = max<int>(max_word_len, filtered.length());
   ```

8. 然后，我们将增加该词在`words map`中的频率。如果是首次处理该单词，那么将会隐式创建一个键值对，然后插入`map`，之后再进行自加操作。

   ```c++
       	++words[filtered];
       }	
   ```

9. 当循环结束时，`words map`会保存所有输入单词的频率。`map`中单词作为键，并且键以字母序排列。我们想要以频率多少进行排序，词频最高的排第一位。为了达到这样的效果，首先实现一个`vector`，将所有键值对放入这个`vector`中。

   ```c++
       vector<pair<string, size_t>> word_counts;
       word_counts.reserve(words.size());
       move(begin(words), end(words), back_inserter(word_counts));
   ```

10. 然后，`vector`中将将具有`words map`中的所有元素。然后，我们来进行排序，把词频最高的单词排在最开始，最低的放在最后。

   ```c++
       sort(begin(word_counts), end(word_counts),
           [](const auto &a, const auto &b) {
           return a.second > b.second;
           });
   ```

11. 现在所有元素如我们想要的顺序排列，之后将这些数据打印在用户的终端上。使用`std::setw`流控制器，可以格式化输出相应的内容。

    ```c++
        cout << "# " << setw(max_word_len) << "<WORD>" << " #<COUNT>\n";
        for (const auto & [word, count] : word_counts) {
            cout << setw(max_word_len + 2) << word << " #"
            	 << count << '\n';
        }
    }
    ```

12. 编译后运行，我们就会得到一个词频表：

    ```c++
    $ cat lorem_ipsum.txt | ./word_frequency_counter
    # <WORD> #<COUNT>
    et #574
    dolor #302
    sed #273
    diam #273
    sit #259
    ipsum #259
    ...
    ```

## How it works...

本节中，我们使用`std::map`实例进行单词统计，然后将`map`中的所有元素放入`vector`中，然后进行排序，再打印输出。为什么要这么做？

先看一个例子。当我们要从`a a b c b b b d c c`字符串中统计词频时，我们的`map`内容如下：

```
a -> 2
b -> 4
c -> 3
d -> 1
```

不过，这是未排序的，这不是我们想要给用户展示的排序。我们的程序要首先输出b的频率，因为b的频率最高。然后是c，a，d。不幸的是，我们无法要求`map`使用键所对应的值进行排序。

这就需要`vector`帮忙了，将`map`中的键值对放入`vector`中。这个方法明确的将这些元素从`map`中删除了。

```c++
vector<pair<string, size_t>> word_counts;
```

然后，我们使用`std::move`函数将词-频对应关系填充整个`vector`。这样的好处是让单词不会重复，不过这样会将元素从`map`中完全删除。使用`move`方法，减少了很多不必要的拷贝。

```c++
move(begin(words), end(words), back_inserter(word_counts));
```

> Note
>
> 一些STL的实现使用短字符优化——当所要处理的字符串过长，这种方法将无需再在堆上分配内存，并且可以将字符串直接进行存储。在这个例子中，移动虽然不是最快的方式，但也不会慢多少。

接下来比较有趣的就是排序操作，其使用了一个Lambda表达式作为自定义比较谓词：

```c++
sort(begin(word_counts), end(word_counts),
	[](const auto &a, const auto &b) { return a.second > b.second; });
```

排序算法将会成对的处理元素，比较两个元素。通过提供的Lambda函数，`sort`方法将不会再使用默认比较谓词，其会将`a.second`和`b.second`进行比较。这里的键值对中，第二个值为词频数，所以可以使用`.second`得到对应词频数。通过这种方式，将移动所有高频率的词到`vector`的开始，并且将低频率词放在末尾。

