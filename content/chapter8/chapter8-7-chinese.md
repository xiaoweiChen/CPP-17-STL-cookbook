# 存储不同的类型——std::variant

C++中支持使用struct和class的方式将不同类型的变量进行包装。当我们想要使用一种类型聊表示多种类型时，我们也可以使用union。不过union的问题在于我们无法知道其是以哪种类型为基础进行的初始化。

看一下下面的代码：

```c++
union U {
    int a;
    char *b;
    float c;
};
void func(U u) { std::cout << u.b << '\n'; }	
```

当我们调用func时，其会将已整型a为基础进行初始化的联合体t进行打印，当然在也无法阻止我们对其他成员进行访问，就像我们使用字符串指针对成员b进行初始化了一样。这段代码会引发各种bug。当我们开始对联合体进行打包之前，有一种辅助变量能够告诉我们其对联合体进行的初始化是安全的，其就是std::variant，在C++17中加入STL。

 variant是一种新的类型，类型安全，并高效的联合体类型。其不使用堆上的内存，所以其是时间和空间都非常高效的，基于联合体的解决方案，这样我们就不用自己再去进行实现了。其能单独存储引用、数组或void类型的成员变量。

本节中，我们将会了解一下由vriant带来的好处。

## How to do it...

让门实现一个程序，其中有两个类型：cat和dog。然后将猫狗混合的存储于一个列表中，这个列表并不具备任何运行时多态性：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <variant>
   #include <list>
   #include <string>
   #include <algorithm>
   
   using namespace std;
   ```

2. 接下来，我们将实现两个具有类似功能的类，不过两个类型之间并没有什么联系。第一个类型是cat。cat对象具有名字，并能喵喵叫：

   ```c++
   class cat {
       string name;
       
   public:
       cat(string n) : name{n} {}
       
       void meow() const {
       	cout << name << " says Meow!\n";
       }
   };
   ```

3. 另一个类是dog。dog对象能汪汪叫：

   ```c++
   class dog {
   	string name;
       
   public:
   	dog(string n) : name{n} {}
       
   	void woof() const {
   		cout << name << " says Woof!\n";
   	}
   };
   ```

4. 那么现在我们就可以来定义一个animal类型，其为`std::variant<dog, cat>`的别名类型。其和旧时代的联合体一样，同时具有variant的新特性：

   ```c++
   using animal = variant<dog, cat>;
   ```

5. 在我们编写主函数之前，我们再来实现两个辅助者。其中之一为动物谓词。通过调用`is_type<cat>(...)`或`is_type<dog>(...)`，我们可以判断动物实例中的动物为cat或dog。其实现只需要对holds_alternative进行调用即可，其为variant类型的一个通用谓词函数：

   ```c++
   template <typename T>
   bool is_type(const animal &a) {
   	return holds_alternative<T>(a);
   }
   ```

6. 第二个辅助者为一个结构器，其看起来像是一个函数对象。其实际是一个双重函数对象，因为其`operator()`实现了两次。一种实现是接受dog作为参数输入，另一个实现是接受cat类型作为参数输入。对于两种实现，其会调用woof或meow函数：

   ```c++
   struct animal_voice
   {
       void operator()(const dog &d) const { d.woof(); }
       void operator()(const cat &c) const { c.meow(); }
   };
   ```

7. 现在让我们使用这些辅助者和类型。首先，我们定义一个animal变量的实例，然后对其进行填充：

   ```c++
   int main()
   {
   	list<animal> l {cat{"Tuba"}, dog{"Balou"}, cat{"Bobby"}};
   ```

8. 现在，我们会将列表的中内容打印三次，并且每次都使用不同的方式。第一种使用`variant::index()`。因为animal类型是`variant<dog, cat>`类型的别名，其返回值的0号索引代表了一个dog的实例。1号索引则代表了cat的实例。变量特化的顺序是这里的关键。在switch-cast代码块中，我们可以通过`get<T>`的方式获取内部的cat或dog实例：

    ```c++
       for (const animal &a : l) {
           switch (a.index()) {
           case 0:
               get<dog>(a).woof();
               break;
           case 1:
               get<cat>(a).meow();
               break;
           }
       }
       cout << "-----\n";
    ```

9. 我们也可以显示的使用类型作为其索引。`get_if<dog>`会返回一个指向dog类型的指针。如果没有dog实例在列表中，那么指针则为null。这样，我们我们可以尝试获取下一种不同类型的实例，直到我们成功为止：

   ```c++
   	for (const animal &a : l) {
           if (const auto d (get_if<dog>(&a)); d) {
           	d->woof();
           } else if (const auto c (get_if<cat>(&a)); c) {
           	c->meow();
           }
       }
       cout << "-----\n";
   ```

10. 使用variant::visit是一种非常优雅的方式。这个函数能够接受一个函数对象，和一个variant实例。函数对象需要对variant中所有可能类型进行重载。我们在之前已经对`operator()`进行了重载，所以这里我们可以直接对其进行使用：

    ```c++
    	for (const animal &a : l) {
    		visit(animal_voice{}, a);
    	}
    	cout << "-----\n";
    ```

11. 最后，我们将回来数一下cat和dog在列表中的数量。`is_type<T>`的cat和dog特化函数，将会与std::count_if结合起来使用，用来返回列表中不同实例的个数：

    ```c++
        cout << "There are "
            << count_if(begin(l), end(l), is_type<cat>)
            << " cats and "
            << count_if(begin(l), end(l), is_type<dog>)
            << " dogs in the list.\n";
    }
    ```

12. 编译并运行程序，我们就会看到打印三次的结果都是相同的。然后，我们可以看到is_type和count_if配合的很不错：

    ```c++
    $ ./variant
    Tuba says Meow!
    Balou says Woof!
    Bobby says Meow!
    -----
    Tuba says Meow!
    Balou says Woof!
    Bobby says Meow!
    -----
    Tuba says Meow!
    Balou says Woof!
    Bobby says Meow!
    -----
    There are 2 cats and 1 dogs in the list.
    ```

## How it works...

std::variant与std::any类型很相似，因为这两个类型都能持有不同类型的变量，并且我们需要在运行时对不同对象进行区分。

另外，std::variant有一个模板列表，需要传入可能在列表中的类型，这点与std::any截然不同。也就是说` std::variant<A, B, C>`必须是A、B或C其中一种实例。当然这也意味着其就不能持有其他类型的变量，除了列表中的类型std::variant没有其他选择。

` variant<A, B, C>`的类型定义，与以下联合体定义类似：

 ``` c++
union U {
    A a;
    B b;
    C c;
};
 ```

在当我们对a, b或c成员变量进行初始化时，联合体中对其进行构建机制需要我们自行区分。std::variant类型就没有这个问题的存在了。

本节的代码中，我们使用了三种方式来处理variant中成员的内容。

首先我们使用了variant的index()成员函数。对变量类型进行索引，`variant<A, B, C>` 中，索引值0代表A类型，1为B类型，2为C类型，以此类推来访问复杂的variant对象。

下一种就是使用`get_if<T>`函数进行获取。其能接受一个variant对象的地址，并且返回一个类型T的指针，指向其内容。如果T类型是错误，那么返回的指针就为null指针。其也可能对varian变量使用`get<T>(x)`来获取对其内容的引用，不过当这样做失败时，函数将会抛出一个异常(在使用get-系列函数进行转换之前，需要使用`holds_alternative<T>(x)`对其类型进行检查)。

最后一种方式就是使用std::visit函数来进行。其能接受一个函数对象和一个variant实例。visit函数会对variant中内容的类型进行检查，然后调用对应的函数对象的重载`operator()`操作符。

为了这个目的，我们实现为了animal_voice类型，因为其将visit和`variant<dog, cat>`类型结合在了一起：

```c++
struct animal_voice
{
    void operator()(const dog &d) const { d.woof(); }
    void operator()(const cat &c) const { c.meow(); }
};
```

以visit的方式对variant进行访问看起来更加的优雅一些，因为使用这种方法就不需要使用硬编码的方式对variant内容中的类型进行判别。这就让我们的代码更加容易扩展。

> Note：
>
> variant类型不能为空的说法并不完全正确。将[std::monostate](http://zh.cppreference.com/w/cpp/utility/variant/monostate)类型添加到其类型列表中，其就能持有空值了。