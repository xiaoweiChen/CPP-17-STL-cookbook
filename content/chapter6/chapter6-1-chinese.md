# 使用STL算法实现单词查找树类

所谓的trie数据类型，能够对感兴趣的数据进行存储，并且易于查找。将文本语句分割成多个单词，放置在列表中，我们能发现其开头一些单词的共性。

让我们看下下图，这里有两个句子“hi how are you”和“hi how do you do”，存储在一个类似于树的结构体中。其都以“hi how”开头，句子后面不同的部分，划分为树结构：

![](../../images/chapter6/6-1-1.png)

因为trie数据结构结合了相同的前缀，其也称为前缀树，很容易使用STL的数据结构实现。本章我们将关注如何实现我们自己的trie类。

## How to do it...

本节，我们将使用STL数据结构和算法实现前缀树结构。

1. 包含必要的头文件和声明所使用的命名空间

   ```c++
   #include <iostream>
   #include <optional>
   #include <algorithm>
   #include <functional>
   #include <iterator>
   #include <map>
   #include <vector>
   #include <string>

   using namespace std;
   ```

2. 我们首先实现一个类。我们的实现中，trie为`map`的递归映射。每个trie节点够包含一个`map`，节点的有效值`T`映射了下一个节点：

   ```c++
   template <typename T>
   class trie
   {
   	map<T, trie> tries;
   ```

3. 将新节点插入队列的代码很简单。使用者需要提供一个`begin/end`迭代器对，并且会通过循环进行递归。当用户输入的序列为{1, 2, 3}时，我们可以将1作为一个子trie，2为下一个子trie，以此类推。如果这些子trie在之前不存在，其将会通过`std::map`的`[]`操作符进行添加：

   ```c++
   public:
       template <typename It>
       void insert(It it, It end_it) {
           if (it == end_it) { return; }
           tries[*it].insert(next(it), end_it);
       }
   ```

4. 我们这里也会定义一个辅助函数，用户只需要提供一个容器，之后辅助函数就会通过迭代器自动进行查询：

   ```c++
   	template <typename C>
       void insert(const C &container) {
       	insert(begin(container), end(container));
       } 
   ```

5. 调用我们的类时，可以写成这样`my_trie.insert({"a", "b","c"}); `，必须帮助编译器正确的判断出这段代码中的所有类型，因此我们又添加了一个函数，这个函数用于重载的插入接口：

   ```c++
   	void insert(const initializer_list<T> &il) {
   		insert(begin(il), end(il));
   	}
   ```

6. 我们也想了解，trie中有什么，所以我们需要一个打印函数。为了打印，我们可以对tire进行深度遍历。这样根节点下面的是第一个叶子节点，我们会记录我们所看到的元素的负载。当我们达到叶子节点，那么就可以进行打印了。我们会看到，当到达叶子的时候`tries.empty()`为true。递归调用print后，我们将再次弹出最后添加的负载元素：

   ```c++
       void print(vector<T> &v) const {
           if (tries.empty()) {
               copy(begin(v), end(v),
               	ostream_iterator<T>{cout, " "});
               cout << '\n';
           }
           for (const auto &p : tries) {
               v.push_back(p.first);
               p.second.print(v);
               v.pop_back();
           }
       }
   ```

7. 打印函数需要传入一个可打印负载元素的列表，不过用户不需要传入任何参数就能调用它。这样，我们就定义了一个无参数的打印函数，其构造了辅助列表对象：

   ```c++
   	void print() const {
           vector<T> v;
           print(v);
       } 
   ```

8. 现在，我们就可以创建和打印trie了，我们将先搜索子trie。当trie包含的序列为`{a, b, c}`和`{a, b, d, e}`，并且我们给定的序列为`{a, b}`，对于查询来说，返回的子序列为包含`{c}`和`{d, e}`的部分。当我们找到子trie，将返回一个`const`的引用。在搜索中，也会出现没有要搜索序列的情况。即便如此，我们还是要返回些什么。`std::optional`是一个非常好的帮手，因为当没有找到匹配的序列，我们可以返回一个空的`optional`对象：

   ```c++
       template <typename It>
       optional<reference_wrapper<const trie>>
       subtrie(It it, It end_it) const {
           if (it == end_it) { return ref(*this); }
           auto found (tries.find(*it));
           if (found == end(tries)) { return {}; }
           
           return found->second.subtrie(next(it), end_it);
       }
   ```

9. 与`insert`方法类似，我们将提供一个只需要一个参数的`subtrie`方法，其能自动的从输入容器中获取迭代器：

   ```c++
       template <typename C>
       auto subtrie(const C &c) {
       	return subtrie(begin(c), end(c));
       }
   };
   ```

10. 这样就实现完了。我们在主函数中使用我们trie类，使用`std::string`类型对类进行特化，并实例化对象：

    ```c++
    int main()
    {
        trie<string> t;
        t.insert({"hi", "how", "are", "you"});
        t.insert({"hi", "i", "am", "great", "thanks"});
        t.insert({"what", "are", "you", "doing"});
        t.insert({"i", "am", "watching", "a", "movie"});
    ```

11. 打印整个trie：

    ```c++
    	cout << "recorded sentences:\n";
    	t.print();
    ```

12. 而后，我们将获取输入语句的子trie，其以“hi”开头：

    ```c++
        cout << "\npossible suggestions after \"hi\":\n";

        if (auto st (t.subtrie(initializer_list<string>{"hi"}));
            st) {
            st->get().print();
        }
    }
    ```

13. 编译并运行程序，其会返回两个句子的以“hi”开头的子trie：

    ```c++
    $ ./trie
    recorded sentences:
    hi how are you
    hi i am great thanks
    i am watching a movie
    what are you doing

    possible suggestions after "hi":
    how are you
    i am great thanks
    ```

## How it works...

有趣的是，单词序列的插入代码要比在子trie查找给定字母序列的代码简单许多。所以，我们首先来看一下插入代码：

```c++
template <typename It>
void trie::insert(It it, It end_it) {
    if (it == end_it) { return; }
    tries[*it].insert(next(it), end_it);
}
```

迭代器对`it`和`end_it`，表示要插入的字符序列。`tries[*it]`代表在子trie中要搜索的第一个字母，然后调用`.insert(next(it), end_it);`对更低级的子trie序列使用插入函数，使用迭代器一个词一个词的推进。` if (it == end_it) { return; }`行会终止递归。返回语句不会做任何事情，这到有点奇怪了。所有插入操作都在`tries[*it]`语句上进行，`std::map`的中括号操作将返回键所对应的值或是创建该键，相关的值(本节中映射类型是一个trie)由默认构造函数构造。这样，当我们查找不理解的单词时，就能隐式的创建一个新的trie分支。

查找子trie看起来十分复杂，因为我们没有必要隐藏那么多的代码：

```c++
template <typename It>
optional<reference_wrapper<const trie>>
subtrie(It it, It end_it) const {
    if (it == end_it) { return ref(*this); }
    auto found (tries.find(*it));
    if (found == end(tries)) { return {}; }

    return found->second.subtrie(next(it), end_it);
}
```

这段代码的主要部分在于`auto found (tries.find(*it));`。我们使用find来替代中括号操作符。当我们使用中括号操作符进行查找时，trie将会为我们创建丢失的元素(顺带一提，当我们尝试这样做，类的函数为`const`，所以这样做事不可能的。这样的修饰能帮助我们减少bug的发生)。

另一个细节是返回值`optional<reference_wrapper<const trie>>`。我们选择`std::optional`作为包装器，因为其可能没有我们所要找打tire。当我们仅插入“hello my friend”，那么就不会找到“goodbye my friend”。这样，我们仅返回`{}`就可以了，其代表返回一个空`optional`对象给调用者。不过这还是没有解释，我们为什么使用`reference_wrapper`代替`optional<const trie &>`。`optional`的实例，其为`trie&`类型，是不可赋值的，因此不会被编译。使用`reference_warpper`实现一个引用，就是用来对对象进行赋值的。