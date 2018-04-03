# 使用constexpr-if简化编译

模板化编程中，通常要以不同的方式做某些事情，比如特化模板类型。C++17带了constexpr-if表达式，可以在很多情况下简化代码。

## How to do it...

本节中，我们会实现一个很小的助手模板类。它能处理不同模板类型的特化，因为他可以在完全不同的代码中，选取相应的片段，依据这些片段的类型对模板进行特化：

1. 完成代码中的通用部分。在我们的例子中，它是一个简单的类，它的成员函数add，支持对U类型值与T类型值的加法:

   ```c++
   template <typename T>
   class addable
   {
     T val;
   public:
     addable(T v) : val{v} {}
     template <typename U>
     T add(U x) const {
       return val + x;
     }
   };
   ```

2. 假设类型T是`std::vector<something>`，而类型U是int。这里就有问题了，为整个vector添加整数是为了什么呢？其应该是对vector中的每个元素加上一个整型数。实现这个功能就需要在循环中进行：

   ```c++
   template <typename U>
   T add(U x)
   {
     auto copy (val); // Get a copy of the vector member
     for (auto &n : copy) {
       n += x;
     }
     return copy;
   }
   ```

3. 下一步也是最后一步，将连接两个世界。



## How it works...





## There's more...



