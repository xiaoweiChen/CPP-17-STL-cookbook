# 自动化管理资源——std::unique_ptr

C++11之后，STL提供了新的智能指针，能对动态内存进行跟踪管理。C++11之前，C++中也有一个智能指针`auto_ptr`，也能对内存进行管理，但是很容易被用错。

不过，使用C++11添加的智能指针的话，我们就很少需要使用到`new`和`delete`操作符。智能指针是自动化内存管理的一个鲜活的例子。当我们使用`unique_ptr`来动态分配对象，基本上不会遇到内存泄漏，因为在析构时会自动的为其所拥有内存使用`delete`操作。

唯一指针表达了其对对象指针的所有权，当对这段内存不在进行使用时，我们会将相关的对象所具有的内存进行释放。这个类将让我们永远远离内存泄漏(智能指针还有`shared_ptr`和`weak_ptr`，不过本节中，我们只关注于`unique_ptr`)。其不会多占用空间，并且不会影响运行时性能，这相较于原始的裸指针和手动内存管理来说十分便捷。(当我们对相应的对象进行销毁后，其内部的裸指针将会被设置为`nullptr`)。

本节中，我们将来看一下`unique_ptr`如何使用。

## How to do it...

我们将创建一个自定义的类型，在构造和析构函数中添加一些调试打印信息，之后展示`unique_ptr`如何对内存进行管理。我们将使用`unique`指针，并使用动态分配的方式对其进行实例化：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <memory>
   
   using namespace std;
   ```

2. 我们将实现一个小类型，后面会使用`unque_ptr`对其实例进行管理。其构造函数和析构函数都会在终端上打印相应的信息，所以之后的自动删除中，我们会看到相应输出的打印：

   ```c++
   class Foo
   {
   public:
       string name;
       
       Foo(string n)
       	: name{move(n)}
       { cout << "CTOR " << name << '\n'; }
       
       ~Foo() { cout << "DTOR " << name << '\n'; }
   };
   ```

3. 为了了解函数对唯一指针在作为参数传入函数的限制，我们可以实现一个这样的函数。其能处理一个`Foo`类型实例，并能将其名称进行打印。注意，`unique`指针是非常智能的，其无额外开销，并且类型安全，也可以为`null`。这就意味着我们仍然要在解引用之前，对指针进行检查：

   ```c++
   void process_item(unique_ptr<Foo> p)
   {
       if (!p) { return; }
       
       cout << "Processing " << p->name << '\n';
   }
   ```

4. 主函数中，我们将开辟一个代码段，在堆上创建两个`Foo`对象，并且使用`unique`指针对内存进行管理。我们显式的使用`new`操作符创建第一个对象实例，并且将其用来创建`unique_ptr<Foo>`变量`p1`。我们通过`make_unique<Foo>`的调用来创建第二个`unique`指针`p2`，我们直接传入参数对`Foo`实例进行构建。这种方式更加的优雅，因为我们使用`auto`类型对类型进行推理，并且能在第一时间对对象进行访问，并且其已经使用`unique_ptr`进行管理：

   ```c++
   int main()
   {
       {
           unique_ptr<Foo> p1 {new Foo{"foo"}};
           auto p2 (make_unique<Foo>("bar"));
       }
   ```

5. 离开这个代码段时，所创建的对象将会立即销毁，并且将内存进行释放。让我们来看一下`process_item`函数和如何使用`unique_ptr`。当创建一个新的`Foo`实例时，其就会被`unique_ptr`进行管理，然后参数的生命周期就在这个函数中。当`process_item`返回时，这个对象就会被销毁：

   ```c++
   	process_item(make_unique<Foo>("foo1"));
   ```

6. 如将已经存在的对象传入`process_item`函数，就需要将指针的所有权进行转移，因为函数需要使用`unique_ptr`作为输入参数，这就会有一次拷贝。但是，`unique_ptr`是无法进行拷贝的，其只能移动。现在让我们来创建两个`Foo`对象，并且将其中一个移动到`process_item`函数中。通过对输出的查阅，我们可以了解到`foo2`在`process_item`返回时会被析构，因为其所有权已经被转移。`foo3`将会持续留存于主函数中，直到主函数返回时才进行析构：

   ```c++
       auto p1 (make_unique<Foo>("foo2"));
       auto p2 (make_unique<Foo>("foo3"));
   
       process_item(move(p1));
   
       cout << "End of main()\n";
   }
   ```

7. 编译并运行程序。首先，我们将看到`foo`和`bar`的构造和析构的输出，离开代码段时就被销毁。我们要注意的是，销毁的顺序与创建的顺序相反。下一个构造的就是`foo1`，其在对`process_item`调用时进行创建。当函数返回时，其就会被立即销毁。然后，我们会创建`foo2`和`foo3`。因为之前转移了指针的所有权，`foo2`会在`process_item`函数调用返回时被立即销毁。另一个元素`foo3`将会在主函数返回时进行销毁：

   ```c++
   $ ./unique_ptr
   CTOR foo
   CTOR bar
   DTOR bar
   DTOR foo
   CTOR foo1
   Processing foo1
   DTOR foo1
   CTOR foo2
   CTOR foo3
   Processing foo2
   DTOR foo2
   End of main()
   DTOR foo3
   ```

## How it works...

使用`std::unique_ptr`来处理堆上分配的对象非常简单。我们初始化`unique`指针之后，其就会指向对应的对象，这样程序就能自动的对其进行释放操作。

当我们将`unique`指针赋予一些新指针时，其就会先删除原先指向的对象，然后再存储新的指针。一个`unique`指针变量`x`，我们可以使用`x.reset()`将其目前所指向的对象进行销毁，然后在指向新的对象。另一种等价方式：`x = new_pointer`与`x.reset(new_pointer)`的方式等价。

> Note：
>
> 的确只有一种方式对`unique_ptr`所指向对象的内存进行释放，那就是使用成员函数`release`，但这种方式并不推荐使用。

解引用之前，需要对指针进行检查，并且其能使用于裸指针相同的方式进行运算。条件语句类似于`if (p){...}`和`if (p != nullptr){...}`，这与我们检查裸指针的方式相同。

解引用一个`unique`指针可以通过`get()`函数完成，其会返回一个指向对应对象的裸指针，并且可以直接进行解引用。

`unique_ptr`有一个很重要的特性——实例无法进行拷贝，只能移动。这就是我们会将已经存在的`unique`指针的所有权转移到`process_item`参数的原因。当我们想要拷贝`unique`指针时，就意味着两个`unique`指针指向相应的对象，这与该指针的设计理念不符，所以`unique`指针对其指向对象的所有权必须唯一。

> Note：
>
> 对于其他的数据类型，由于智能指针的存在，所以很少使用`new`和`delete`对其进行手动操作。尽可能的使用智能指针！特别是`unqiue_ptr`，其在运行时无任何额外开销。



