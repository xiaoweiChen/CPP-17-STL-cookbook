# 迭代器填充容器——std::istream

上节中，我们学习了如何从输入流中向数据结构中读入数据，然后用这些数据填充列表或向量。

这次，我们将使用标准输入来填充`std::map`。问题在于我们不能将一个结构体进行填充，然后从后面推入到线性容器中，例如`lis`t和`vector`，因为`map`的负载分为键和值两部分。

完成本节后，我们会了解到如何从字符流中将复杂的数据结构进行序列化和反序列化。

## How to do it...

本节，我们会定义一个新的结构体，不过这次将其放入`map`中，这会让问题变得复杂，因为容器中使用键值来表示所有值。

1. 包含必要的头文件，并声明所使用的命名空间。

   ```c++
   #include <iostream>
   #include <iomanip>
   #include <map>
   #include <iterator>
   #include <algorithm>
   #include <numeric>
   
   using namespace std;
   ```

2. 我们会引用网络上的一些梗。这里的梗作为一个名词，我们记录其描述和诞生年份。我们会将这些梗放入`std::map`，其名称为键，包含在结构体中的其他信息作为值：

   ```c++
   struct meme {
       string description;
       size_t year;
   };
   ```

3. 我们暂时先不去管键，我们先来实现结构体`meme`的流操作符`operator>>`。我们假设相关梗的描述由双引号括起来，后跟对应年份。举个栗子，` "some description" 2017`。通过使用`is >> quoted(m.description)`，双引号会被当做限定符，直接被丢弃。这就非常的方便。然后我们继续读取年份即可：

   ```c++
   istream& operator>>(istream &is, meme &m) {
   	return is >> quoted(m.description) >> m.year;
   }
   ```

4. OK，现在将梗的名称作为键插入`map`中。为了实现插入`map`，需要一个`std::pair<key_type, value_type>`实例。`key_type`为`string`，那么`value_type`就是`meme`了。名字中允许出现空格，所以可以使用`quoted`对名称进行包装。`p.first`是名称，`p.second`代表的是相关`meme`结构体变量。可以使用`operator>>`实现直接对其进行赋值：

   ```c++
   istream& operator >>(istream &is,
   				    pair<string, meme> &p) {
   	return is >> quoted(p.first) >> p.second;
   }
   ```

5. 现在来写主函数，创建一个`map`实例，然后对其进行填充。因为对流函数`operator>>`进行了重载，所以可以直接对`istream_iterator`类型直接进行处理。我们将会从标准输入中解析出更多的信息，然后使用`inserter`迭代器将其放入`map`中：

   ```c++
   int main()
   {
       map<string, meme> m;
       
       copy(istream_iterator<pair<string, meme>>{cin},
      		{},
       	inserter(m, end(m))); 
   ```

6. 对梗进行打印前，先在`map`中找到名称最长的梗吧。可以对其使用`std::accumulate`。累加的初始值为0u(u为无符号类型)，然后逐个访问`map`中的元素，将其进行合并。使用`accumulate`合并，就意味着叠加。例子中，并不是对数值进行叠加，而是对最长字符串的长度进行进行累加。为了得到长度，我们为`accumulate`提供了一个辅助函数`max_func`，其会将当前最大的变量与当前梗的名字长度进行比较(这里两个数值类型需要相同)，然后找出这些值中最大的那个。这样`accumulate`函数将会返回当前梗中，名称最长的梗：

   ```c++
       auto max_func ([](size_t old_max,
       				 const auto &b) {
       	return max(old_max, b.first.length());
       });
       size_t width {accumulate(begin(m), end(m),
       					    0u, max_func)};
   ```

7. 现在，对`map`进行遍历，然后打印其中每一个元素。使用`<< left << setw(width)`打印出漂亮的“表格”：

   ```c++
       for (const auto &[meme_name, meme_desc] : m) {
           const auto &[desc, year] = meme_desc;
           
           cout << left << setw(width) << meme_name
                << " : " << desc
                << ", " << year << '\n';
       }
   }
   ```

8. 现在需要一些梗的数据，我们写了一些梗在文件中：

   ```c++
   "Doge" "Very Shiba Inu. so dog. much funny. wow." 2013
   "Pepe" "Anthropomorphic frog" 2016
   "Gabe" "Musical dog on maximum borkdrive" 2016
   "Honey Badger" "Crazy nastyass honey badger" 2011
   "Dramatic Chipmunk" "Chipmunk with a very dramatic look" 2007
   ```

9. 编译并运行程序，将文件作为数据库进行输入：

   ```c++
   $ cat memes.txt | ./filling_containers
   Doge: Very Shiba Inu. so dog. much funny. wow., 2013
   Dramatic Chipmunk : Chipmunk with a very dramatic look, 2007
   Gabe: Musical dog on maximum borkdrive, 2016
   Honey Badger: Crazy nastyass honey badger, 2011
   Pepe: Anthropomorphic frog, 2016
   ```

## How it works...

本节有三点需要注意。第一，没有选择`vector`或`list`比较简单的结构，而是选择了`map`这样比较复杂的结构。第二，使用了`quoted`控制符对输入流进行处理。第三，使用`accumulate`来找到最长的键值。

我们先来看一下`map`，结构体`meme`只包含一个`description`和`year`。因为我们将梗的名字作为键，所以没有将其放入结构体中。可以将`std::pair`实例插入`map`中，首先实现了结构体`meme`的流操作符`operator>>`，然后对`pair<string, meme>`做同样的事。最后，使用` istream_iterator<pair<string, meme>>{cin}`从标准输入中获取每个元素的值，然后使用` inserter(m, end(m)) `将组对插入`map`中。

当我们使用流对`meme`元素进行赋值时，允许梗的名称和描述中带有空格。我们使用引号控制符，很轻易的将问题解决，得到的信息类似于这样，`"Name with spaces" "Description with spaces" 123`。

当输入和输出都对带有引号的字符串进行处理时，`std::quoted`就能帮助到我们。当有一个字符串`s`，使用`cout << quoted(s)`对其进行打印，将会使其带引号。当对流中的信息进行解析时，`cin >> quoted(s)`其就能帮助我们将引号去掉，保留引号中的内容。

叠加操作是对`max_func`的调用看起来很奇怪：

```c++
auto max_func ([](size_t old_max, const auto &b) {
	return max(old_max, b.first.length());
});

size_t width {accumulate(begin(m), end(m), 0u, max_func)};
```

实际上，`max_func`能够接受一个`size_t`和一个`auto`类型的参数，这两个参数将转换成一个`pair`，从而就能插入`map`中。这看起来很奇怪，因为二元函数会将两个相同类型的变量放在一起操作，例如`std::plus`。我们会从每个组对中获取键值的长度，将当前元素的长度值与之前的最长长度相对比。

叠加调用会将`max_func`的返回值与0u值进行相加，然后作为左边参数的值与下一个元素进行比较。第一次左边的参数为0u，所以就可以写成`max(0u, string_length)`，这时返回的值就作为之前最大值，与下一个元素的名称长度进行比较，以此类推。