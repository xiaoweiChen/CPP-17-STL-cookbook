# 迭代器填充容器——std::istream

在上节中，我们学习了如何从输入流中向数据结构中读入数据，然后用这些填充列表或向量。

这次，我们将使用标准输入来填充std::map。问题在于我们不能将一个结构体进行填充，然后从后面推入到线性容器中，例如list和vector，因为map的负载分为键和值两部分。我们将会看到其不会完全不同。

在了解本节后，我们了解到如何从字符流中将复杂的数据结构进行序列化和反序列化。

## How to do it...

本节我们会定义一个新的结构体，不过这次我们将其放入map中，这会让问题变得复杂，因为容器中使用键值来表示所有值。

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

2. 我们会引用网络上的一些梗。这里的梗作为一个名词，我们记录其描述和诞生年份。我们会将这些梗放入std::map，其名称为键，包含在结构体中的其他信息作为值：

   ```c++
   struct meme {
       string description;
       size_t year;
   };
   ```

3. 我们暂时先不去管键，我们先来实现结构体meme的流操作符`operator>>`。我们假设相关梗的描述由双引号括起来，后跟对应年份。举个栗子，` "some description" 2017`。通过使用`is >> quoted(m.description)`，双引号会被当做限定符，直接被丢弃。这就非常的方便。然后我们继续读取年份即可：

   ```c++
   istream& operator>>(istream &is, meme &m) {
   	return is >> quoted(m.description) >> m.year;
   }
   ```

4. OK，现在让我们将梗的名称作为键插入map中。为了实现插入map，我们需要一个`std::pair<key_type, value_type>`实例。key_type为string，那么value_type就是meme了。名字中允许出现空格，所以我么可以使用quoted对名称进行包装。p.first是名称，p.second代表的是相关meme结构体变量。我们可以使用`operator>>`实现直接对其进行赋值：

   ```c++
   istream& operator >>(istream &is,
   				    pair<string, meme> &p) {
   	return is >> quoted(p.first) >> p.second;
   }
   ```

5. 现在让我们来写主函数，创建一个map实例，然后对其进行填充。因为我们对流函数`operator>>`进行了重载，所以可以直接对istream_iterator类型直接进行处理。我们将会从标准输入中解析出更的信息，然后使用inserter迭代器将其放入map中：

   ```c++
   int main()
   {
       map<string, meme> m;
       
       copy(istream_iterator<pair<string, meme>>{cin},
      		{},
       	inserter(m, end(m))); 
   ```

6. 在我们对梗进行打印前，让我们先在map中找到名称最长的梗吧。可以对其使用std::accumulate。累加的初始值为0u(u为无符号类型)，然后我们逐个访问map中的元素，将其进行合并。使用accumulate合并，就意味着叠加。我们的例子中，并不是对数值进行叠加，而是对最长字符串的长度进行进行累加。为了得到长度，我们为accumulate提供了一个辅助函数max_func，其会将当前最大的变量与当前梗的名字长度进行比较(这里两个数值类型需要相同)，然后找出这些值中最大的那个。这样accumulate函数将会返回当前梗中，名称最长的梗：

   ```c++
       auto max_func ([](size_t old_max,
       				 const auto &b) {
       	return max(old_max, b.first.length());
       });
       size_t width {accumulate(begin(m), end(m),
       					    0u, max_func)};
   ```

7. 现在，我们对map进行遍历，然后打印其中每一个元素。我们使用`<< left << setw(width)`打印出漂亮的“表格”：

   ```c++
       for (const auto &[meme_name, meme_desc] : m) {
           const auto &[desc, year] = meme_desc;
           
           cout << left << setw(width) << meme_name
                << " : " << desc
                << ", " << year << '\n';
       }
   }
   ```

8. 现在我们需要一些梗的数据，我们写了一些梗在文件中：

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

本节有三点需要注意。第一，我们没有选择vector或list比较简单的结构，而是选择了map这样比较复杂的结构。第二，我们使用了quoted控制符对输入流进行处理。第三，我们使用accumulate来找到最长的键值。

我们先来看一下map。我们的结构体meme只包含一个description和year。因为我们将梗的名字作为键，所以没有将其放入结构体重。我们可以将std::pair实例插入map中。我们也就是这样做的。我们首先实现了结构体mem的流操作符`operator>>`，然后我们对`pair<string, meme>`做同样的事。而后，我们使用` istream_iterator<pair<string, meme>>{cin}`从标准输入中获取每个元素的值，然后使用` inserter(m, end(m)) `将组对插入map中。

当我们使用流对meme元素进行赋值时，我们允许梗的名称和描述中带有空格。我们使用引号控制符，很轻易的将问题解决，得到的信息类似于这样，`"Name with spaces" "Description with spaces" 123`。

当输入和输出都对带有引号的字符串进行处理时，std::quoted就能很好的帮助我们。当我们有一个字符串s，使用`cout << quoted(s)`对其进行打印，将会使其带引号。当我们对流中的信息进行解析时，`cin >> quoted(s)`其就能帮助我们将引号去掉，保留引号中的内容。

叠加操作是对max_func的调用看起来很奇怪：

```c++
auto max_func ([](size_t old_max, const auto &b) {
	return max(old_max, b.first.length());
});

size_t width {accumulate(begin(m), end(m), 0u, max_func)};
```

实际上，max_func能够接受一个size_t和一个auto类型的参数，这两个参数将转换成一个pair，从而就能插入map中。这看起来很奇怪，因为二元函数会将两个相同类型的变量放在一起操作，例如std::plus。我们会从每个组对中获取键值的长度，将当前元素的长度值与之前的最长长度相对比。

在叠加调用会将max_func的返回值与0u值进行相加，然后作为左边参数的值与下一个元素进行比较。第一次左边的参数为0u，所以就可以写成`max(0u, string_length)`，这时返回的值就作为之前最大值，与下一个元素的名称长度进行比较，以此类推。