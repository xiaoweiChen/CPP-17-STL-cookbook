# 将标准算法进行组合

gather为STL算法中最好的组合性例子。Sean Parent在任Adobe系统主要科学家时，就在向世人普及这个算法，因为其本身短小精悍。其使用方式就如同做一件艺术品一样。

gather算法能操作任意的元素类型。其更改元素的顺序，通过用户的选择，其会将对应的数据放置在对应位置上。

## How to do it...

本节，我们将来实现gather算法，并对其进行一定的修改。最后，我们将展示如何进行使用：

1. 包含必要的头文件，声明所使用的命名空间。

   ```c++
   #include <iostream>
   #include <algorithm>
   #include <string>
   #include <functional>
   
   using namespace std; 
   ```

2. gather算法是表现标准算法组合性很好的一个例子。gather接受一对begin/end迭代器，和另一个迭代器gather_pos，其指向begin和end迭代器的中间位置。最后一个参数是一个谓词函数。使用谓词函数，算法会将满足谓词条件的所有元素放置在gather_pos迭代器附近。我们使用std::stable_partition来完成移动元素的任务。gather算法将会返回一对迭代器。这两个迭代器由stable_partition所返回，其表示汇集范围的起始点和结束点：

   ```c++
   template <typename It, typename F>
   pair<It, It> gather(It first, It last, It gather_pos, F predicate)
   {
   	return {stable_partition(first, gather_pos, not_fn(predicate)),
   		    stable_partition(gather_pos, last, predicate)};
   }
   ```

3. 算法的另一种辩题为gather_sort。其工作原理与gather相同，不过其不接受一元谓词函数；其能接受一个二元比较函数。这样，其就也能将对应值汇集在gather_pos附近，并且能知道其中最大值和最小值：

   ```c++
   template <typename It, typename F>
   void gather_sort(It first, It last, It gather_pos, F comp_func)
   {
       auto inv_comp_func ([&](const auto &...ps) {
       	return !comp_func(ps...);
       });
       
       stable_sort(first, gather_pos, inv_comp_func);
       stable_sort(gather_pos, last, comp_func);
   }
   ```

4. 让我们来使用一下这些算法。我们先创建一个谓词函数，其会告诉我们当前的字母是不是a。我们再构建一个字符串，仅包含‘a’和‘-’字符：

   ```c++
   int main()
   {
       auto is_a ([](char c) { return c == 'a'; });
       string a {"a_a_a_a_a_a_a_a_a_a_a"};
   ```

5. 继续构造一个迭代器，让其指向字符串的中间位置。这时我们可以调用gather算法，然后看看会发生什么。‘a’字符将汇集在字符串中间的某个位置附近：

   ```c++
       auto middle (begin(a) + a.size() / 2);
       
   	gather(begin(a), end(a), middle, is_a);
       cout << a << '\n';
   ```

6. 再次调用gather，不过这次gather_pos的位置在字符串的起始端：

   ```c++
   	gather(begin(a), end(a), begin(a), is_a);
   	cout << a << '\n';
   ```

7. 再将gather_pos的位置放在末尾试试：

   ```c++
   	gather(begin(a), end(a), end(a), is_a);
   	cout << a << '\n';
   ```

8. 最后一次，这次将再次将迭代器指向中间位置。这次与我们的期望不相符，后面我们来看看发生了什么：

   ```c++
   	// This will NOT work as naively expected
   	gather(begin(a), end(a), middle, is_a);
   	cout << a << '\n';
   ```

9. 我们再构造另一个字符串，使用下划线和一些数字组成。对于这个输入队列，我们使用gather_sort。gather_pos迭代器指向字符串的中间，并且比较函数为`std::less<char>`：

   ```c++
       string b {"_9_2_4_7_3_8_1_6_5_0_"};
       gather_sort(begin(b), end(b), begin(b) + b.size() / 2,
       		   less<char>{});
       cout << b << '\n';
   }
   ```

10. 编译并运行程序，我们就会看到如下的输出。对于前三行来说，和我们的预期一样，不过第四行貌似出现了一些问题。最后一行中，我们可以看到gather_short函数的结果。数字的顺序是排过序的(中间小，两边大)：

    ```c++
    $ ./gather
    _____aaaaaaaaaaa_____
    aaaaaaaaaaa__________
    __________aaaaaaaaaaa
    __________aaaaaaaaaaa
    _____9743201568______
    ```

## How it works...

gather算法有点难以掌握，因为其非常短，但处理的是比较复杂的问题。让我们来逐步解析这个算法：

![](../../images/chapter6/6-7-1.png)

1. 初始化相应元素，并提供一个谓词函数。图中满足谓词函数条件的元素为灰色，其他的为白色。迭代器a和c表示整个范围的长度，并且迭代器b指向了最中间的元素。这就表示要将所有灰色的格子聚集在这个迭代器附近。
2. gather算法会对范围(a, b]调用std::stable_partition，并对另一边使用不满足谓词条件的结果。这样就能将所有灰色格子集中在b迭代器附近。与我们预期相反的事情发生了。
3. 另一个std::stable_partition已经完成，不过在[b, c)间我们将使用满足谓词函数的结果。这样就将灰色的格子汇集在b迭代器附近。
4. 所有灰色的格子都汇集在b迭代器附近，这是就可以返回起始迭代器x和末尾迭代器y，用来表示所有连续灰色格子的范围。

我们对同一个范围多次调用gather算法。最初，我们将所有元素放在范围中间位置。然后，我们尝试放在开始和末尾。这种实验很有趣，因为这会让其中一个std::stable_partition无元素可以处理。

在我们最后一次对gather进行调用时，参数为(begin, end, middle)，但没有和我们预期的一样，这是为什么呢？首先，这看起来像是一个bug，实际上不是。

试想一个字符组"aabb"，使用谓词函数is_character_a，用来判断元素是否为'a'，当我们将第三个迭代器指向字符范围的中间时，我们会复现这个bug。原因是，第一个stable_partition调用会对子范围"aa"进行操作，并且另一个stable_partition会对子范围"bb"上进行操作。这种串行的调用时无法得到"baab"的，其结果看起来和开始一样，没有任何变化。

> Note：
>
> 要是想得到我们预期的序列，我们可以使用`std::rotate(begin, begin + 1, end);`

gather_sort基本上和gather差不多。签名不同的就是在于谓词函数。实现的不同在于gather调用了两次std::stable_partition，而gather_sort调用了两次std::stable_sort。

这是由于not_fn不能作用域二元函数，所以反向比较不能由not_fn完成。