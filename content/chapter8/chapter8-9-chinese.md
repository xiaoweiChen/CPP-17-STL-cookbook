# 处理共享堆内存——std::shared_ptr

上一节中，我们了解了如何使用unique_ptr。这个类型非常有用，因为其能帮助我们管理动态分配的对象。不过，其所有权只能让一个类型对象所有。其不能让多个对象指向同一个动态分配的对象，因为如果这样的话它就不知道要对那个对象进行删除了。

指针类型shared_ptr就是为了应对这种情况所设计的。共享指针可以随时进行拷贝。其内部有一个计数器，记录了有多少对象持有这个指针。只有当最后一个持有者被销毁时，其才会对动态分配的对象进行删除。同样，其也不会让我陷入内存泄漏的窘境，因为对象也会在我们使用之后进行自动删除。同时，我们需要确定对象没有过早的被删除，或是删除的过于频繁(每次对象的创建都要进行一次删除)。

本节中，你将了解到如何使用shared_ptr自动的对动态对象进行管理，并且能在多个所有者间共享动态对象，而后了解其与unique_ptr之间的区别：

## How to do it...

我们将完成一个与unique_ptr节类似的程序，以展示shared_ptr的用法：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <memory>
   
   using namespace std; 
   ```

2. 然后定义一个辅助类，其能帮助我们了解类何时创建和销毁。我们将会使用shared_ptr对内存进行管理：

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

3. 接下来，我们将实现一个函数Foo，其参数的类型为共享指针。接受共享指针作为参数的方式，要比引用有意思的多，因为这样我们不会进行拷贝，但是会改变共享这指针内部的计数器：

   ```c++
   void f(shared_ptr<Foo> sp)
   {
       cout << "f: use counter at "
       	<< sp.use_count() << '\n';
   }
   ```

4. 主函数中，我们声明一个空的共享指针。通过默认构造方式对其进行构造，其实际上是一个null指针：

   ```c++
   int main()
   {
   	shared_ptr<Foo> fa;
   ```

5. 下一步，我么将创建一个代码段，并创建两个Foo对象。我们使用new操作符对第一个对象进行创建，然后使用构造函数在shared_ptr中创建这一对象。我们直接使用`make_shared<Foo>`对第二个实例进行创建，其使用我们提供的参数创建一个Foo实例。这种创建的方式很优雅，我们使用auto类型进行类型推断，这样对象也就算是被第一次访问到。这里与unique_ptr很类似：

   ```c++
   	{
           cout << "Inner scope begin\n";
           
           shared_ptr<Foo> f1 {new Foo{"foo"}};
           auto f2 (make_shared<Foo>("bar"));
   ```

6. 当共享指针被共享时



## How it works...



![](../../images/chapter8/8-9-1.png)



## There's more...



