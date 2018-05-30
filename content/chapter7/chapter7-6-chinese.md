# 格式化输出

很多情况下，我们仅对打印字符串和数字是不够的。数字通常都以十进制进行打印，而有时我们需要使用十六进制或八进制进行打印。并且在打印十六进制的时候，我们希望看到以"0x"为前缀的十六进制的数字，但有时候却不希望看到这个前缀。

当对浮点数进行打印的时候，也需要注意很多。以何种精度进行打印？要将数中的所有内容进行打印吗？或者是如何打印科学计数法样式的数？

除了数值表示方面的问题外，我们还需要规范我们打印的格式。有时我们要以表格的方式进行打印，以确保打印出的数据的可读性。

这所有的一切都与输出流有关。有时对输入流的解析也是十分重要的。本节中我们将来感受一下格式化输出。有些显示也会比较棘手，所以我们会对其进行一些解释。

## How to do it...

为了让大家熟悉格式化输出，本节我们将使用各种各样的格式进行打印：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <iomanip>
   #include <locale>

   using namespace std;
   ```

2. 接下来，我们定义一个辅助函数，其会以不同的方式打印出一个数值。其能接受使用一种字符对宽度进行填充，其默认字符为空格：

   ```c++
   void print_aligned_demo(int val,
                           size_t width,
                           char fill_char = ' ')
   { 
   ```

3. 使用setw，我们可以设置打印数字的最小字符数输出个数。当我们要将123的输出宽度设置为6时，我们会得到"abc   "或"   abc"。我们也可以使用std::left, std::right和std::internal控制从哪边进行填充。当我们以十进制的方式对数字进行输出，internal看起来和right的作用一样。不过，当打印0x1时，打印宽度为6时，internal会得到"0x  6"。setfill控制符可以用来定义填充字符。我么可以尝试使用使用以下方式进行打印：

   ```c++
       cout << "================\n";
       cout << setfill(fill_char);
       cout << left << setw(width) << val << '\n';
       cout << right << setw(width) << val << '\n';
       cout << internal << setw(width) << val << '\n';
   }
   ```

4. 主函数中，我们使用已经实现的函数。首先，我们打印数字12345，其宽度为15。我们进行两次打印，不过第二次时，我们将填充字符设置为'_'：

   ```c++
   int main()
   {
       print_aligned_demo(123456, 15);
       print_aligned_demo(123456, 15, '_');
   ```

5. 随后，我们将打印0x123abc，并使用同样的宽度。不过，打印之前需要使用的是std::hex和std::showbase告诉输出流对象cout输出的格式，并且添加"0x"前缀，让其看起来是一下十六进制数：

   ```c++
   	cout << hex << showbase;
   	print_aligned_demo(0x123abc, 15); 
   ```

6. 对于八进制我们也可以做同样的事：

   ```c++
   	cout << oct;
   	print_aligned_demo(0123456, 15);
   ```

7. 通过hex和uppercase，我们可以将"0x"中的x转换成大写字母。"0x123abc"中的"abc"同样也转换成大写：

   ```c++
   	cout << "A hex number with upper case letters: "
   		<< hex << uppercase << 0x123abc << '\n';	
   ```

8. 如果我们要以十进制打印100，我们需要将输出从hex切换回dec：

   ```c++
       cout << "A number: " << 100 << '\n';
       cout << dec;
       
   	cout << "Oops. now in decimal again: " << 100 << '\n';
   ```

9. 我们可以对布尔值的输出进行配置，通常，true会打印出1，false为0。使用boolalpha，我们就可以得到文本表达：

   ```c++
   	cout << "true/false values: "
   		<< true << ", " << false << '\n';
   	cout << boolalpha
   		<< "true/false values: "
   		<< true << ", " << false << '\n';
   ```

10. 现在让我们来一下浮点型变量float和double的打印。当我们有一个数12.3，那么打印也应该是12.3。当我们有一个数12.0，打印时会将小数点那一位进行丢弃，不过我们可以通过showpoint来控制打印的精度。使用这个控制符，就能显示被丢弃的一位小数了：

    ```c++
        cout << "doubles: "
            << 12.3 << ", "
            << 12.0 << ", "
            << showpoint << 12.0 << '\n';
    ```

11. 可以使用科学计数法或固定浮点的方式来表示浮点数。scientific会将浮点数归一化成一个十进制的小数，并且其后面的位数使用10的幂级数表示，其需要进行乘法后才能还原成原始的浮点数。比如，300.0科学计数法就表示为"3.0E2"，因为300 = 3.0 x 10^2。fixed将会恢复普通小数的表达方式：

    ```c++
    	cout << "scientific double: " << scientific
    		<< 123000000000.123 << '\n';
    	cout << "fixed double: " << fixed
    		<< 123000000000.123 << '\n';
    ```

12. 除此之外，我们也能对打印的精度进行控制。我们先创建一个特别小的浮点数，并对其小数点后的位数进行控制：

    ```c++
    	cout << "Very precise double: "
    		<< setprecision(10) << 0.0000000001 << '\n';
    	cout << "Less precise double: "
    		<< setprecision(1) << 0.0000000001 << '\n';
    }
    ```

13. 编译并运行程序，我们就会得到如下的输出。前四个块都是有打印辅助函数完成，其使用setw对字符串进行了不同方向的填充。此外，我们也进行了数字的进制转换、布尔数表示和浮点数表示。通过使用，我们会对其更加熟悉：

    ```c++
    $ ./formatting
          ================
          123456
    123456
    123456
          ================
          123456_________
          _________123456
          _________123456
          ================
          0x123abc
    0x123abc
          0x 123abc
          ================
          0123456
    0123456
    0123456
          A hex number with upper case letters: 0X123ABC
          A number: 0X64
          Oops. now in decimal again: 100
          true/false values: 1, 0
          true/false values: true, false
          doubles: 12.3, 12, 12.0000
          scientific double: 1.230000E+11
          fixeddouble: 123000000000.123001
          Very precise double: 0.0000000001
          Less precise double: 0.0
    ```

## How it works...

所有这些看起来有些长，并且`<< foo << bar`的方式对于这块不会很了解的读者来说会感觉到困惑。因此，让我们来看一下格式化修饰符的表。其都是用`input_stream >> modifier `或` output_stream << modifier`来对之后的输入输出进行影响：

| 符号                                       | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| setprecision(int)                          | 打印浮点数时，决定打印小数点后的位数。                       |
| showpoint / noshowpoint                    | 启用或禁用浮点数字小数点的打印，即使没有小数位。             |
| fixed /scientific / hexfloat /defaultfloat | 数字可以以固定格式和科学表达式的方式进行打印。fixed和scientific代表了相应的打印模式。hexfloat将会同时激活这两种模式，用十六进制浮点表示法格式化浮点数。defaultfloat则会禁用这两种模式。 |
| showpos / noshowpos                        | 启用或禁用使用'+'来标志正浮点数。                            |
| setw(int n)                                | 设置打印的宽度n。在读取的时候，这种设置会截断输入。当打印位数不够时，其会使用填充字符将输出填充到n个字符。 |
| setfill(char c)                            | 当我们setw时，会涉及填充字符的设置。setfill可以将填充字符设置为c。其默认填充字符为空格。 |
| internal / left / right                    | left和right控制填充的方向。internal会将填充字符放置在数字和符号之间，这对于十六进制打印和一些金融数字来说，十分有用。 |
| dec / hex / oct                            | 整数打印的类型，十进制、十六进制和八进制。                   |
| setbase(int n)                             | 数字类型的同义函数，当n为10/16/8时，与dec / hex / oct完全相同。当传入0时，则会恢复默认输出，也就是十进制，或者使用数字的前缀对输入进行解析。 |
| quoted(string)                             | 将带有引号的字符串的引号去掉，对其实际字符进行打印。这里string的类型可以是string类的实例，也可以是一个C风格的字符串。 |
| boolalpha / noboolalpha                    | 打印布尔变量，是打印字符形式的，还是数字形式的。             |
| showbase / noshowbase                      | 启用或禁用基于前缀的数字解析。对于hex来说就是0x，对于octal来说就是0。 |
| uppercase / nouppercase                    | 启用或禁用将浮点数中的字母或十六进制中的字符进行大写输出。   |

看起来很多，想要熟悉这些控制符的最好方式，还是尽可能多的使用它们。