# std::unordered_map中使用自定义类型

当我们使用`std::unordered_map`代替`std::map`时，对于键的选择要从另一个角度出发。`std::map`要求键的类型可以排序。因此，元素可以进行排序。不过，当我们使用数学中的向量作为键呢？这样一来就没有判断哪个向量大于另一个向量，比如向量(0, 1)和(1, 0)无法相比较，因为它们指向的方向不同。在`std::unordered_map`中这都不是问题，因为不需要对键的哈希值进行排序。对于我们来说只要为类型实现一个哈希函数和等同`==`操作符的实现，等同操作符的是实现是为了判断两个对象是否完全相同。本节中，我们就来实验一下这个例子。

## How to do it...

本节中，我们要定义一个简单的`coord`数据结构，其没有默认哈希函数，所以我们必须要自行定义一个。然后我们会使用`coord`对象来对应一些值。

1. 包含使用`std::unordered_map`所必须的头文件

   ```c++
   #include <iostream>
   #include <unordered_map> 
   ```

2. 自定义数据结构，这是一个简单的数据结构，还不具备对应的哈希函数：

   ```c++
   struct coord {
   	int x;
   	int y;
   };
   ```

3. 实现哈希函数是为了能让类型作为键存在，这里先实现比较操作函数：

   ```c++
   bool operator==(const coord &l, const coord &r)
   {
   	return l.x == r.x && l.y == r.y;
   }
   ```

4. 为了使用STL哈希的能力，我们打开了std命名空间，并且创建了一个特化的`std::hash`模板。其使用`using`将特化类型进行别名。

   ```c++
   namespace std
   {
   template <>
   struct hash<coord>
   {
       using argument_type = coord;
       using result_type = size_t;
   ```

5. 下面要重载该类型的括号表达式。我们只是为`coord`结构体添加数字，这是一个不太理想的哈希方式，不过这里只是展示如何去实现这个函数。一个好的散列函数会尽可能的将值均匀的分布在整个取值范围内，以减少哈希碰撞。

   ```c++
       result_type operator()(const argument_type &c) const
       {
           return static_cast<result_type>(c.x)
          		   + static_cast<result_type>(c.y);
       }
   };
   }
   ```

6. 我们现在可以创建一个新的`std::unordered_map`实例，其能结构`coord`结构体作为键，并且对应任意值。例子中对`std::unordered_map`使用自定义的类型来说，已经很不错了。让我们基于哈希进行实例化，并填充自定义类型的`map`表，并打印这个`map`表：

   ```c++
   int main()
   {
       std::unordered_map<coord, int> m { 
           { {0, 0}, 1}, 
           { {0, 1}, 2},
           { {2, 1}, 3}
       };
       for (const auto & [key, value] : m) {
           std::cout << "{(" << key.x << ", " << key.y
       			 << "): " << value << "} ";
       }
       std::cout << '\n';
   }
   ```

7. 编译运行这个例子，就能看到如下的打印信息：

   ```
   $ ./custom_type_unordered_map
   {(2, 1): 3} {(0, 1): 2} {(0, 0): 1}
   ```

## How it works...

通常实例化一个基于哈希的map表(比如: `std::unordered_map`)时，我们会这样写：

```c++
std::unordered_map<key_type, value_type> my_unordered_map;
```

编译器为我们创建特化的`std::unordered_map`时，这句话背后隐藏了大量的操作。所以，让我们来看一下其完整的模板类型声明：

```c++
template<
    class Key,
    class T,
    class Hash = std::hash<Key>,
    class KeyEqual = std::equal_to<Key>,
    class Allocator = std::allocator< std::pair<const Key, T> >
> class unordered_map;
```

这里第一个和第二个模板类型，我么填写的是`coord`和`int`。另外的三个模板类型是选填的，其会使用已有的标准模板类。这里前两个参数需要我们给定对应的类型。

对于这个例子，`class Hash`模板参数是最有趣的一个：当我们不显式定义任何东西时，其就指向`std::hash<key_type>`。STL已经具有`std::hash`的多种特化类型，比如`std::hash<std::string>`、`std::hash<int>`、`std::hash<unique_ptr>`等等。这些类型中可以选择最优的一种类型类解决对应的问题。

不过，STL不知道如何计算我们自定义类型`coord`的哈希值。所以我们要使用我们定义的类型对哈希模板进行特化。编译器会从`std::hash`特化列表中，找到我们所实现的类型，也就是将自定义类型作为键的类型。

如果新特化一个`std::hash<coord>`类型，并且将其命名成my_hash_type，我们可以使用下面的语句来实例化这个类型：

```c++
std::unordered_map<coord, value_type, my_hash_type> my_unordered_map;
```

这样命名就很直观，可读性好，而且编译器也能从哈希实现列表中找到与之对应的正确的类型。