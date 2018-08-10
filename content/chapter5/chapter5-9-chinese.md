# 生成输入序列的序列

当测试代码需要处理参数顺序不重要的输入序列时，有必要测试它是否对所有可能的输入产生相同的输出。当你自己实现了一个排序算法时，就要写这样的测试代码来确定自己的实现是否正确。

`std::next_permutation`在任何时候都能帮我们将序列进行打乱。我们在可修改的范围中可以调用它，其会将以字典序进行置换。

## How to do it...

本节，我们将从标准输入中读取多个字符串，然后使用`std::next_permutation`生成已排序的序列，并且打印这个序列：

1. 首先，包含必要的头文件，并声明所使用的命名空间。

   ```c++
   #include <iostream>
   #include <vector>
   #include <string>
   #include <iterator>
   #include <algorithm>
   
   using namespace std; 
   ```

2. 使用标准数组对`vector`进行初始化，接下来对`vector`进行排序：

   ```c++
   int main()
   {
       vector<string> v {istream_iterator<string>{cin}, {}};
       sort(begin(v), end(v));
   ```

3. 现在来打印`vector`中的内容。随后，调用`std::next_permutation`，其会打乱已经排序的`vector`，再对其进行打印。直到`next_permutation`返回false时，代表`next_permutation`完成了其操作，循环结束：

   ```c++
       do {
           copy(begin(v), end(v),
           	ostream_iterator<string>{cout, ", "});
           cout << '\n';
       } while (next_permutation(begin(v), end(v)));
   }
   ```

4. 编译运行这个程序，会有如下的打印：

   ```c++
   $ echo "a b c" | ./input_permutations
   a, b, c,
   a, c, b,
   b, a, c,
   b, c, a,
   c, a, b,
   c, b, a,
   ```

## How it works...

`std::next_permutation`算法使用起来有点奇怪。因为这个函数接受一组开始/结束迭代器，当其找到下一个置换时返回true；否则，返回false。不过“下一个置换”又是什么意思呢？

当`std::next_permutation`算法找到元素中的下一个字典序时，其会以如下方式工作：

1. 通过`v[i - 1] < v[i]`的方式找到最大索引i。如果这个最大索引不存在，那么返回false。
2. 再找到最大所以j，这里j需要大于等于i，并且`v[j] > v[i - 1]`。
3. 将位于索引位置j和i - 1上的值进行交换。
4. 将从i到范围末尾的元素进行反向。
5. 返回true。

每次单独的置换顺序，都会在同一个序列中呈现。为了看到所有置换的可能，我们先对数组进行了排序。如果我们输入“c b a”到算法中，算法会立即终止，因为每个元素都以反字典序排列。



