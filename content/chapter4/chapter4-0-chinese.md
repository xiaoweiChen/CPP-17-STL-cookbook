# 第4章 Lambda表达式

Lambda表达式是C++11添加的非常重要的一个特性。C++14和C++17对Lambda进行补充，使得Lambda表达式如虎添翼。那就先了解一下，什么是Lambda表达式呢？

Lambda表达式或者Lambda函数为闭包结构。闭包是描述未命名对象的通用术语，也可以称为匿名函数。为了在C++中加入这个特性，就需要相应对象实现`()`括号操作符。C++11之前想要实现类似具有Lambda的对象，代码如下所示：

```c++
#include <iostream>
#include <string>
int main() {
    struct name_greeter {
        std::string name;
        
        void operator()() {
        	std::cout << "Hello, " << name << '\n';
        }
    };
    
    name_greeter greet_john_doe {"John Doe"};
    greet_john_doe();
}
```

构造`name_greeter`对象需要传入一个字符串。这里需要注意的是，这个结构类型，Lambda可以使用一个没有名字的实例来表示。对于闭包结构来说，我们称之为捕获一个字符串。其就像我们在构造这个例子中的实例时传入的字符串一样，不过Lambda不需要参数，就能完成打印`Hello, John Doe`。

C++11之后，使用闭包的方式来实现会更加简单：

```c++
#include <iostream>
int main() {
    auto greet_john_doe ([] {
    	std::cout << "Hello, John Doe\n";
    });
    greet_john_doe();
}
```

这样就行了！不再需要`name_greeter`结构体，直接使用Lambda表达式替代。这看起来像魔术一样，本章的第一节中会对细节进行详细的描述。

Lambda表达式对于完成通用和简介类代码是非常有帮助的。其能对通用的数据结构进行处理，这样就不惧用户指定的特殊类型。闭包结构也会被用来将运行在线程上的数据进行打包。C++11标准推出后，越来越多的库支持了Lambda表达式，因为这对于C++来说已经是很自然的事情了。另一种使用方式是用于元编程，因为Lambda在编译时是可以进行预估的。不过，我们不会往元编程的方向去讲述，元编程的内容可能会撑爆这本书。

本章我们着重于函数式编程，对于那些对函数式编程不了解的开发者或初学者来说，这看起来非常的神奇。如果你在代码中看到Lambda表达式横飞，请先别沮丧。在这个函数式编程越来越流行的年代，需要拓展对于现代C++的了解。如果你看到的代码有点复杂，建议你多花点时间去分析它们。当你驯服了Lambda表达式，你就能驾驭它驰骋疆场，不再会为之困惑。

