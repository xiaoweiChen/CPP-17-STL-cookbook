# 从用户的输入读取数值

本书中大多数例程的输入都是从文件或标准输入中获得。这次我们重点来了解一下读取，以及当遇到一些有问题的流时，不能直接终止程序，而是要做一些错误处理的工作。

本节只会从用户输入中读取，知道如何读取后，将了解如何从其他的流中读取数据。用户输入通常通过`std::cin`，其为最基础的输入流对象，类似这样的类还有`ifstream`和`istringstream`。

## How to do it...

本节，将从用户输入中读取不同值，并且了解如何进行错误处理，并对输入中有用的部分进行较为复杂的标记。

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>

   using namespace std;
   ```

2. 首先，提示用户输入两个数字。将这两个数字解析为`int`和`double`类型。例如，用户输入`1 2.3`:

   ```c++
   int main()
   {
       cout << "Please Enter two numbers:\n> ";
       int x;
       double y;
   ```

3. 解析和错误检查同时在`if`判断分支中进行。只有两个数都被解析成有效的数字，才能对其进行打印：

   ```c++
       if (cin >> x >> y) {
       	cout << "You entered: " << x
       		 << " and " << y << '\n';
   ```

4. 如果因为任何原因，解析不成功，那么我们要告诉用户为什么会出错。`cin`流对象现在处于失败的状态，并且将错误状态进行清理之前，无法为我们提供输入功能。为了能够对新的输入进行解析需要调用`cin.clear()`，并且将之前接受到的字符丢弃。使用`cin.ignore`完成丢弃的任务，这里我们指定了丢弃字符的数量，直到遇到下一个换行符为止。完成这些事之后，输入有可以用了：

   ```c++
       } else {
           cout << "Oh no, that did not go well!\n";
           cin.clear();
           cin.ignore(
           	std::numeric_limits<std::streamsize>::max(),
           	'\n');
       }
   ```

5. 让用户输入一些其他信息。我们让用户输入名字，名字由多个字母组成，字母间使用空格隔开。因此，可以使用`std::getline`函数，其需要传入一个流对象和一个分隔字符。我们选逗号作为分隔字符。这里使用`getline`来代替`cin >> ws`的方式读入字符，这样我们就能丢弃在名字前的所有空格。对于每一个循环中都会打印当前的名字，如果名字为空，那么我们会将其丢弃：

   ```c++ 
       cout << "now please enter some "
       		"comma-separated names:\n> ";
       for (string s; getline(cin >> ws, s, ',');) {
       	if (s.empty()) { break; }
       	cout << "name: \"" << s << "\"\n";
       }
   }
   ```

6. 编译并运行程序，就会得到如下的输出，其会让用户进行输入，然后我们输入合法的字符。数字`1 2`都能被正确的解析，并且后面输入的名字也能立即排列出来。两个逗号间没有单词的情况将会跳过：

   ```c++
   $ ./strings_from_user_input
   Please Enter two numbers:
   > 1 2
   You entered: 1 and 2
   now please enter some comma-separated names:
   > john doe,ellen ripley, alice,chuck norris,,
   name: "john doe"
   name: "ellen ripley"
   name: "alice"
   name: "chuck norris"
   ```

7. 再次运行程序，这次将在一开始就输入一些非法数字，可以看到程序就会走到不同的分支，然后丢弃相应的输入，并继续监听正确的输入。可以看到`cin.clear()`和`cin.ignore(...)`的调用如何对名字读取进行影响：

   ```c++
   $ ./strings_from_user_input
   Please Enter two numbers:
   > a b
   Oh no, that did not go well!
   now please enter some comma-separated names:
   > bud spencer, terence hill,,
   name: "bud spencer"
   name: "terence hill"
   ```

## How it works...

本节，我们对一些复杂输入进行了检索。首要注意的是，我们的检索和错误处理是同时进行。

表达式`cin >> x`是对`cin`的再次引用。因此，就可以将输入些为`cin >> x >> y >> z >> ...`。与此同时，其也能将输入内容转换成为一个布尔值，并在`if`条件中使用。这个布尔值告诉我们最后一次读取是否成功，这也就是为什么我们会将代码写成`if (cin >> x >> y) { ... }`的原因。

当我们想要读取一个整型，但输入中包含`foobar`为下一个表示，那么流对象将无法对这段字符进行解析，并且这让输入流的状态变为失败。这对于解析来说是非常关键的，但对于整个程序来说就不是了。这里可以将输入流的状态进行重置，然后在进行其他的操作。在我们的例程中，我们尝试在读取两个数值失败后，读取一组姓名。例子中，我们使用`cin.clear()`对`cin`的工作状态进行了重置。不过，这样内部的光标就处于我们的现在的类型上，而非之前的数字。为了将之前输入的内容丢弃，并对姓名输入进行流水式的读取，我们使用了一个比较长的表达式，`  cin.ignore(std::numeric_limits<std::streamsize>::max(),'\n');`。这里对内存的清理是十分有必要的，因为我们需要在用户输入一组姓名时，对缓存进行刷新。

下面的循环看起来也挺奇怪的：

```c++
for (string s; getline(cin >> ws, s, ',');) { ... }
```

`for`循环的判断部分，使用了`getline`函数。`getline`函数接受一个输入流对象，一个字符串引用作为输出，以及一个分隔符。通常，分隔字符代表新的一行。这里使用逗号作为分隔符，所以姓名输入列表为`john, carl, frank`，这样就可以单个的进行读取。

目前为止还不错。不过，`cin >> ws`的操作是怎么回事呢？这可以让`cin`对所有空格进行刷新，其会读取下一个非空格字符到下一个逗号间的字符。回看一下"john, carl, frank"例子，当我们不使用`ws`时，将获取到"john"，" carl"和" frank"字符串。这里需要注意"carl"和"frank"开头不必要的空格，因为在`ws`中对输入流进行了预处理，所以能够避免开头出现空格的情况。