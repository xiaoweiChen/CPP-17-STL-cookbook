# 自动化管理资源——std::unique_ptr

在C++11之后，STL提供了新的智能指针，其能对动态内存进行跟踪管理。在C++11之前，C++中也有一个智能指针auto_ptr，其也能对内存进行管理，但是其很容易被用错。

不过，使用C++11添加的智能指针的话，我们就很少需要使用到new和delete操作符。智能指针是自动化内存管理的一个鲜活的例子。当我们使用unique_ptr来动态分配对象，那我们基本上与内存泄漏无缘了，因为其在析构的时候回自动的为其所拥有内存使用delete操作。

唯一指针表达了其对对象指针的所有权，当我们对这段内存不在进行使用时，我们会将相关的对象所具有的内存进行释放。这个类将让我们永远的远离内存泄漏(智能指针还有shared_ptr和weak_ptr，不过本节中，我们只关注于unique_ptr)。其在不会多占用空间，并且不会影响运行时性能，这与原始的裸指针和手动内存管理来说，是十分方便的。(Okay，当我们对相应的对象进行销毁后，其内部的裸指针将会被设置为nullptr，这样其就不总能被优化。不过通过手写的代码和对动态内存的管理，也有同样的问题)。

本节中，我们将来看一下unique_ptr如何使用。

## How to do it...

我们将创建一个自定义的类型，然后在这个类型的构造和析构函数中添加一些调试打印信息，之后展示unique_ptr如何对内存进行管理。我么将使用unique指针，并使用动态分配的方式对其进行实例化：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <memory>
   
   using namespace std;
   ```

2. 我们将实现一个小类型，后面会使用unque_ptr对其实例进行管理。其构造函数和析构函数都会在终端上打印相应的信息，所以我们在之后可以看到会在之后自动的删除：

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

3. 为了了解函数对唯一指针在作为参数传入函数的限制，我们就来实现一个这样的函数。其能处理一个Foo类型实例，并能将其名称进行打印。注意，unique指针是非常智能的，其无额外开销，并且类型安全，也可以为null。这也就意味着我们仍然要在解引用之前，对指针进行检查：

   ```c++
   void process_item(unique_ptr<Foo> p)
   {
       if (!p) { return; }
       
       cout << "Processing " << p->name << '\n';
   }
   ```

4. 主函数中，我们将开辟一个代码段，在堆上创建两个Foo对象，并且使用unique指针对内存进行管理。我们显式的使用new操作符创建第一个对象实例，并且将其用来创建`unique_ptr<Foo>`变量p1。我们通过`make_unique<Foo>`的调用来创建第二个unique指针p2，我们直接传入参数对Foo实例进行构建。这种方式更加的优雅，因为我们使用auto类型对类型进行推理，并且我们能在第一时间对对象进行访问，并且其已经使用unique_ptr进行管理：

   ```c++
   int main()
   {
       {
           unique_ptr<Foo> p1 {new Foo{"foo"}};
           auto p2 (make_unique<Foo>("bar"));
       }
   ```

5. 当我们离开这个代码段时，所创建的对象将会被立即销毁，并且将内存进行释放。让我们来看一下process_item函数和如何使用unique_ptr。当我们创建一个新的Foo实例时，其就会被unique_ptr进行管理，然后参数的生命周期就在这个函数中。当process_item返回时，这个对象就会被销毁：

   ```c++
   	process_item(make_unique<Foo>("foo1"));
   ```

6. 如将已经存在的对象传入process_item函数时，我们就需要将指针的所有权进行转移，因为函数需要使用unique_ptr作为输入参数，这就意味着将会有一次拷贝。但是，unique_ptr是无法进行拷贝的，其只能被移动。现在让我们来说创建两个Foo对象，并且将其中一个移动到process_item函数中。通过对输出的查阅，我么可以了解到foo2在process_item返回时会被析构，因为其所有权已经被转移。foo3将会持续留存于主函数中，直到主函数返回时才进行析构：

   ```c++
       auto p1 (make_unique<Foo>("foo2"));
       auto p2 (make_unique<Foo>("foo3"));
   
       process_item(move(p1));
   
       cout << "End of main()\n";
   }
   ```

7. 编译并运行程序。首先，我们将看到foo和bar的构造和析构的输出。其在离开代码段时，就被销毁。我们要注意的是，销毁的顺序与创建的顺序相反。下一个构造的就是foo1，其在对process_item调用时进行创建。当函数返回时，其就会被立即销毁。然后，我们会创建foo2和foo3。foo2会在process_item函数调用返回时被立即销毁，因为我们在之前转移了指针的所有权。另一个元素foo3将会在主函数返回时进行销毁：

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

使用std::unique_ptr来处理堆上分配的对象非常简单。在我们初始化unique指针之后，其就会指向对应的对象，这样程序就能自动的对其进行释放操作。

当我们将unique指针赋予一些新指针时，其就会先删除原先指向的对象，然后再存储新的指针。一个unique指针变量x，我们可以使用x.reset()将其目前所指向的对象进行销毁，然后在指向新的对象。另一种等价方式：`x = new_pointer`与`x.reset(new_pointer)`的方式等价。

> Note：
>
> 的确只有一种方式对unique_ptr所指向对象的内存进行释放，那就是使用成员函数release，但这种方式并不推荐使用。

在解引用之前，指针需要进行检查，并且其能使用于裸指针相同的方式进行运算。条件语句类似于`if (p){...}`和`if (p != nullptr){...}`，这与我们检查裸指针的方式相同。

解引用一个unique指针可以通过get()函数完成，其会返回一个指向对应对象的裸指针，并且可以直接进行解引用。

unique_ptr有一个很重要的特性，就是其实力无法进行拷贝，只能进行移动。这也就是为什么我们会将已经存在的unique指针的所有权转移个process_item参数的原因。当我们想要拷贝unique指针时，其就意味着对应对象被两个unique指针所指向，这与该指针的设计原理不符，unique指针对其指向对象的所有权必须唯一。

> Note：
>
> 对于其他的数据类型，由于智能指针的存在，所以很少使用new和delete对其进行手动操作。尽可能的使用智能指针！特别是unqiue_ptr，其在运行时无任何额外开销。



