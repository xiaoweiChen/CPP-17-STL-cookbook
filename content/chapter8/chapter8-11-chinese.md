# 使用智能指针简化处理遗留API

智能指针(`unique_ptr`，`shared_ptr`和`weak_ptr`)非常有用，并且对于开发者来说，可以使用其来代替手动分配和释放空间。

当有对象不能使用`new`操作进行创建，或不能使用`delete`进行释放呢？过去有很多库都有自己的分配和释放函数。这看起来好像是个问题，因为我么了解的智能指针都依赖于`new`和`delete`。那么如何在智能指针中，使用指定的工厂函数对特定类型的对象进行创建或是销毁呢？

这个问题一点都不难。本节中，我们将来了解一下如何为智能指针指定特定的分配器和销毁器。

## How to do it...

本节中，我们将定义一种不能使用`new`创建的类型，并且也不能使用`delete`进行释放。对于这种限制，我们依旧选择直接使用智能指针，这里使用`unique_ptr`和`shared_ptr`实例来进行演示。

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <memory>
   #include <string>
   
   using namespace std; 
   ```

2. 声明一个类，将其构造函数和析构函数声明为`private`。我们使用这样的方式来模拟无法直接和销毁对象实例的情况：

   ```c++
   class Foo
   {
       string name;
       
       Foo(string n)
       	: name{n}
       { cout << "CTOR " << name << '\n'; }
       
       ~Foo() { cout << "DTOR " << name << '\n';}
   ```

3. 然后，声明两个静态函数`create_foo`和`destroy_foo`，这两个函数用来对`Foo`实例进行创建和销毁，其会对裸指针进行操作。这是用来模拟使用旧C风格的API，这样我们就不能用之前的方式直接对`shared_ptr`指针进行使用：

   ```c++
   public:
       static Foo* create_foo(string s) {
       	return new Foo{move(s)};
       }
   
       static void destroy_foo(Foo *p) { delete p; }
   };
   ```

4. 现在，我们用`shared_ptr`来对这样的对象进行管理。对于共享指针，我们可以通过`create_foo`函数来构造相应的对象。只有销毁的方式有些问题，因为`shared_ptr`默认的销毁方式会有问题。解决方法就是我们将自定义的销毁器给予`shared_ptr`。删除函数或删除可调用对象的函数签名需要需要与`destroy_foo`函数统一。当我们的删除函数非常复杂，那我们可以使用Lambda表达式对其进行包装：

   ```c++
   static shared_ptr<Foo> make_shared_foo(string s)
   {
   	return {Foo::create_foo(move(s)), Foo::destroy_foo};
   }
   ```

5. 需要注意的是`make_shared_foo`函数，将会返回一个普通的`shared_ptr<Foo>`实例，因为设置了自定义的销毁器，并不会对其类型有所影响。从编程角度上，之前是因为`shared_ptr`调用了虚函数，将设置销毁器的步骤隐藏了。唯一指针(`unique_ptr`)不会带来任何额外开销，所以这种方式不适合唯一指针。目前，我们就需要对`unique_ptr`所持有的类型进行修改。我们将`void(*)(Foo*)`类型作为第二个模板参数传入，其也就是`destroy_foo`函数的类型：

   ```c++
   static unique_ptr<Foo, void (*)(Foo*)> make_unique_foo(string s)
   {
   	return {Foo::create_foo(move(s)), Foo::destroy_foo};
   }
   ```

6. 主函数中，我们直接使用函数对两个智能指针进行实例化。程序的输出中，我们将看到相应的对象会被创建，然后自动销毁：

   ```c++
   int main()
   {
       auto ps (make_shared_foo("shared Foo instance"));
       auto pu (make_unique_foo("unique Foo instance"));
   }
   ```

7. 编译并运行程序，我们就会得到如下输出，输出与我们的期望一致：

   ```c++
   $ ./legacy_shared_ptr
   CTOR shared Foo instance
   CTOR unique Foo instance
   DTOR unique Foo instance
   DTOR shared Foo instance
   ```

## How it works...

通常来说，当`unique_ptr`和`shared_ptr`要销毁其持有的对象时，只会对内部指针使用`delete`。本节中，我们的类无法使用C++常用的方式进行创建和销毁。`Foo::create_foo`函数会返回一个构造好的`Foo`指针，这对于智能指针来说没什么，因为指针指针也可以对裸指针进行管理。

其问题在于，当对象不能使用默认方式删除时，如何让`unique_ptr`和`shared_ptr`接触到对象的析构函数。

在这方面，两种智能指针有些不同。为了为`unique_ptr`设置一个自定义销毁器，我们需要对其类型进行修改。因为`Foo`的销毁函数为` void Foo::destroy_foo(Foo*); `，那么`unique_ptr`所是有`Foo`的类型必须为` unique_ptr<Foo, void(*)(Foo*)> `。现在，`unique_ptr`也就获取了`destroy_foo`的指针了，在`make_unique_foo`函数中其作为构造的第二个模板参数传入。

`unique_ptr`为了自定义销毁器函数，需要对持有类型进行修改，那么为什么`shared_ptr`就不需要呢？我们也能向`shared_ptr`的第二个模板参数传入对应的类型的呀。为什么`shared_ptr`的操作就要比`unique_ptr`简单呢？

这是因为`shared_ptr`支持可调用删除器对象，而不用影响共享指针做指向的类型，这种功能在控制块中进行。共享指针的控制块是一个对象的虚函数。这也就意味着标准共享指针的控制块，与给定了自定义的销毁器的共享指针的控制块不同！当我们要让一个唯一指针使用一个自定义销毁器时，就需要改变唯一指针所指向的类型。当我们想让共享指针使用自定义销毁器时，只需要对内部控制块的类型进行修改即可，这种修改的过程对我们是不可见的，因为其不同隐藏在虚函数的函数接口中。

当然，我们可以手动的为`unique_ptr`做发生在`shared_ptr`上的事情，不过这会增加运行时的开销。这是我们所不希望看到的，因为`unique_ptr`能够保证在运行时无任何额外开销。