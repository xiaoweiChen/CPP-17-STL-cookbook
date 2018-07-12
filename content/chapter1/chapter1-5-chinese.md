# 使用constexpr-if简化编译

模板化编程中，通常要以不同的方式做某些事情，比如特化模板类型。C++17带了`constexpr-if`表达式，可以在很多情况下简化代码。

## How to do it...

本节中，我们会实现一个很小的辅助模板类。它能处理不同模板类型的特化，因为它可以在完全不同的代码中，选取相应的片段，依据这些片段的类型对模板进行特化：

1. 完成代码中的通用部分。在我们的例子中，它是一个简单的类，它的成员函数`add`，支持对`U`类型值与`T`类型值的加法:

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

2. 假设类型`T`是`std::vector<something>`，而类型`U`是`int`。这里就有问题了，为整个`vector`添加整数是为了什么呢？其应该是对`vector`中的每个元素加上一个整型数。实现这个功能就需要在循环中进行：

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

3. 下一步也是最后一步，将两种方式结合在一起。如果`T`类型是一个`vector`，其每个元素都是`U`类型，择进行循环。如果不是，则进行普通的加法：

   ```c++
   template <typename U>
   T add(U x) const{
       if constexpr(std::is_same<T, std::vector<U>>::value){
           auto copy(val);
           for (auto &n : copy){
               n += x;
           }
           return copy;
       } else {
           return val + x;
       }
   }
   ```

4. 现在就可以使用这个类了。让我们来看下其对不同类型处理的是多么完美，下面的例子中有`int`,` float`, `std::vector<int>`和`std::vector<string>`:

   ```c++
   addable<int> {1}.add(2); // is 3
   addable<float> {1.f}.add(2); // is 3.0
   addable<std::string> {"aa"}.add("bb"); // is "aabb"

   std::vector<int> v{1, 2, 3};
   addable<std::vector<int>> {v}.add(10); // is std::vector<int> {11, 12, 13}

   std::vector<std::string> sv{"a", "b", "c"};
   addable<std::vector<std::string>> {sv}.add(std::string{"z"}); // is {"az", "bz", "cz"}
   ```


## How it works...

新特性`constexpr-if`的工作机制与传统的`if-else`类似。不同点就在于前者在编译时进行判断，后者在运行时进行判断。所以，使用`constexpr-if`的代码在编译完成后，程序的这一部分其实就不会有分支存在。有种方式类似于`constexpr-if`，那就是`#if-#else`的预编译方式进行宏替换，不过这种方式在代码的构成方面不是那么优雅。组成`constexpr-if`的所有分支结构都是优雅地，没有使用分支在语义上不要求合法。

为了区分是向`vector`的每个元素加上x，还是普通加法，我们使用`std::is_same`来进行判断。表达式`std::is_same<A, B>::value`会返回一个布尔值，当A和B为同样类型时，返回true，反之返回false。我们的例子中就写为`std::is_same<T, std::vector<U>>::value()`(`is_same_v = is_same<T, U>::value;`)，当返回为true时，且用户指定的T为`std::vector<X>`，之后试图调用add，其参数类型`U = X`。

当然，在一个`constexpr-if-else`代码块中，可以有多个条件(注意：a和b也可以依赖于模板参数，并不需要其为编译时常量)：

```c++
if constexpr(a){
    // do something
} else if constexpr(b){
    // do something else
} else {
    // do something completely different
}
```

C++17中，很多元编程的情况更容易表达和阅读。

## There's more...

这里对比一下C++17之前的实现和添加`constexpr-if`后的实现，从而体现出这个特性的加入会给C++带来多大的提升：

```c++
template <typename T>
class addable{
    T val;
public:
    addable(T v):val{v}{}
    
    template <typename U>
    std::enable_if_t<!std::is_same<T, std::vector<U>>::value, T> 
    add(U x) const {
        return val + x;
    }
    
    template <typename U>
    std::enable_if_t<!std::is_same<T, std::vector<U>>::value, std::vector<U>>
    add (U x) const{
        auto copy(val);
        for (auto &n: copy){
            n += x;
        }
        return copy;
    }
};
```

在没有了`constexpr-if`的帮助下，这个类看起特别复杂，不像我们所期望的那样。怎么使用这个类呢？

简单来看，这里重载实现了两个完全不同的`add`函数。其返回值的类型声明，让这两个函数看起里很复杂；这里有一个简化的技巧——表达式，例如`std::enable_if_t<condition, type>`，如果条件为真，那么就为`type`类型，反之`std::enable_if_t`表达式不会做任何事。这通常被认为是一个错误，不过我们能解释为什么什么都没做。

对于第二个`add`函数，相同的判断条件，但是为反向。这样，在两个实现不能同时为真。

当编译器看到具有相同名称的不同模板函数并不得不选择其中一个时，一个重要的原则就起作用了：替换失败不是错误([SFINAE](http://zh.cppreference.com/w/cpp/language/sfinae), **Substitution Failure is not An  Error**)。这个例子中，就意味着如果函数的返回值来源一个错误的模板表示，无法推断得出，这时编译器不会将这种情况视为错误(和`std::enable_if`中的条件为false时的状态一样)。这样编译器就会去找函数的另外的实现。

很麻烦是吧，C++17中实现起来就变得简单多了。