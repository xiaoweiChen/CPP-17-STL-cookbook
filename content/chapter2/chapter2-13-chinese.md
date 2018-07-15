# 实现个人待办事项列表——std::priority_queue

`std::priority_queue`是另一种适配容器(类似于`std::stack`)。其实为另一种数据结构的包装器(默认的数据结构为`std::vector`)，并且提供类似队列的接口。同样也遵循队列的特性，先进先出。这与我们之前使用的`std::stack`完全不同。

这里仅仅是对`std::queue`的行为进行描述，本节将展示`std::priority_queue`是如何工作的。这个适配器比较特殊，其不仅有FIFO的特性，还混合着优先级。这就意味着，FIFO的原则会在某些条件下被打破，根据优先级的顺序形成子FIFO队列。

## How to do it...

本节中，我们将创建一个待办事项的结构。为了程序的简明性就不从用户输入解析输入了。这次专注于`std::priority_queue`的使用。所以我们使用一些待办事项和优先级填充一个优先级序列，然后以FIFO的顺序读出这些元素(这些元素是通过优先级进行过分组)。

1. 包含必要的头文件。`std::priority_queue`在`<queue>`中声明。

   ```c++
   #include <iostream>
   #include <queue>
   #include <tuple>
   #include <string>
   ```

2. 我们怎么将待办事项存在优先级队列中呢？我们不能添加项目时，附加优先级。优先级队列将使用自然序对待队列中的所有元素。现在我们实现一个自定义的结构体`struct todo_item`，并赋予其优先级系数，和一个字符串描述待办事件，并且为了让该结构体具有可排序性，这里会实现比较操作符`<`。另外，我们将会使用`std::pair`，其能帮助我们聚合两个类型为一个类型，并且能完成自动比较。

   ```c++
   int main()
   {
   	using item_type = std::pair<int, std::string>;
   ```

3. 那么现在我们有了一个新类型`item_type`，其由一个优先级数字和一个描述字符串构成。所以，我们可以使用这种类型实例化一个优先级队列。

   ```c++
   	std::priority_queue<item_type> q;
   ```

4. 我们现在来填充优先级队列。其目的就是为了提供一个非结构化列表，之后优先级队列将告诉我们以何种顺序做什么事。比如，你有漫画要看的同时，也有作业需要去做，那么你必须先去写作业。不过，`std::priority_queue`没有构造函数，其支持初始化列表，通过列表我们能够填充优先级队列(使用`vector`或`list`都可以对优先级队列进行初始化)。所以我们这里定义了一个列表，用于下一步的初始化。

   ```c++
       std::initializer_list<item_type> il {
           {1, "dishes"},
           {0, "watch tv"},
           {2, "do homework"},
           {0, "read comics"},
       };
   ```

5. 现在我们可以很方便的遍历列表中的所有元素，然后通过`push`成员函数将元素插入优先级列表中。

   ```c++
       for (const auto &p : il) {
       	q.push(p);
       }
   ```

6. 这样所有的元素就都隐式的进行了排序，并且我们可以浏览列表中优先级最高的事件。

   ```c++
       while(!q.empty()) {
           std::cout << q.top().first << ": " << q.top().second << '\n';
           q.pop();
       }
       std::cout << '\n';
   }
   ```

7. 编译运行程序。结果如我们所料，作业是最优先的，看电视和看漫画排在最后。

   ```c++
   $ ./main
   2: do homework
   1: dishes
   0: watch tv
   0: read comics
   ```

## How it works...

`std::priority_queue`使用起来很简单。我们只是用了其三个成员函数。

1. `q.push(item)`将元素推入队列中。
2. `q.top()`返回队首元素的引用。
3. `q.pop()`移除队首元素。

不过，如何做到排序的呢？我们将优先级数字和描述字符串放入一个`std::pair`中，然后就自然得到排序后的结果。这里有一个` std::pair<int, std::string>`的实例`p`，我们可通过`p.first`访问优先级整型数，使用`p.second`访问字符串。我们在循环中就是这样打印所有待办事件的。

如何让优先级队列意识到` {2, "do homework"}`要比`{0, "watch tv"}`重要呢？

比较操作符`<`在这里处理了不同的元素。我们假设现在有`left < right`，两个变量的类型都是pair。

1. ` left.first != right.first`，然后返回`left.first < right.first`。
2. ` left.first == right.first`，然后返回`left.second < right.second`。

以这种方式就能满足我们的要求。最重要的就是`pair`中第一个成员，然后是第二个成员。否则，`std::priority_queue`将会字母序将元素进行排序，而非使用数字优先级的顺序(这样的话，看电视将会成为首先要做的事情，而完成作业则是最后一件事。对于懒人来说，无疑是个完美的顺序)。

