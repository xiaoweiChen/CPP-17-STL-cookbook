# 使用输入文件初始化复杂对象

将整型、浮点型和字符串分开读取不是困难，因为流操作符`>>`对于基础类型有重载的版本，并且输入流会将输入中的空格去除。

不过，对于更加复杂的结构体来说，我们应该如何将其从输入流中读取出来，并且当我们的字符串中需要多个单词的时候应该怎么做呢(在空格处不断开)？

对于任意类型，我们都可以对输入流`operator>>`操作符进行重载，接下来我们就要看下如何做这件事：

## How to do it...

本节，我们将定义一个数据结构，并从标准输入中获取数据：

1. 包含必要的头文件和声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <iomanip>
   #include <string>
   #include <algorithm>
   #include <iterator>
   #include <vector>
   
   using namespace std; 
   ```

2. 创建一个复杂的对象，我们定义了一个名为`city`的结构体。城市需要有名字，人口数量和经纬坐标。

   ```c++
   struct city {
       string name;
       size_t population;
       double latitude;
       double longitude;
   };
   ```

3. 为了从输入流中读取一个城市的信息，这时我们就需要对`operator>>`进行重载。对于操作符来说，会跳过`ws`开头的所有空格，我们不希望空格来污染城市的名称。然后，会对一整行的文本进行读取。这样类似于从输入文件中读取一整行，行中只包含城市的信息。然后，我们就可以用空格将人口，经纬度进行区分：

   ```c++
   istream& operator>>(istream &is, city &c)
   {
       is >> ws;
       getline(is, c.name);
       is >> c.population
           >> c.latitude
           >> c.longitude;
       return is;
   }
   ```

4. 主函数中，我们创建一个`vector`，其包含了若干城市元素，使用`std::copy`将其进行填充。我们会将输入的内容拷贝到`istream_iterato`中。通过给定的`city`结构体作为模板参数，其会使用重载过的`operator>>`进行数据的读取：

   ```c++
   int main()
   {
       vector<city> l;
       
       copy(istream_iterator<city>{cin}, {},
       	back_inserter(l)); 
   ```

5. 为了了解城市信息是否被正确解析，我们会将其进行打印。使用格式化输出`left << setw(15) <<`，城市名称左边必有很多的空格，这样我们的输出看起来就很漂亮：

    ```c++
       for (const auto &[name, pop, lat, lon] : l) {
           cout << left << setw(15) << name
               << " population=" << pop
               << " lat=" << lat
               << " lon=" << lon << '\n';
       }
      }
    ```

6. 例程中所用到的文件内容如下。我们将四个城市的信息写入文件：

   ```c++
   Braunschweig
   250000 52.268874 10.526770
   Berlin
   4000000 52.520007 13.404954
   New York City
   8406000 40.712784 -74.005941
   Mexico City
   8851000 19.432608 -99.133208
   ```

7. 编译并运行程序，将会得到如下输入。我们在输入文件中为城市名称前添加一些不必要的空白，以查看空格是如何被过滤掉的：

   ```c++
   $ cat cities.txt| ./initialize_complex_objects
   Braunschweig    population = 250000 lat = 52.2689 lon = 10.5268
   Berlin          population = 4000000 lat = 52.52 lon = 13.405
   New York City   population = 8406000 lat = 40.7128 lon = -74.0059
   Mexico City     population = 8851000 lat = 19.4326 lon = -99.1332
   ```

## How it works...

本节也非常短。我们只是创建了一个新的结构体`city`，我们对`std::istream`迭代器的`operator>>`操作符进行重载。这样也就允许我们使用`istream_iterator<city>`对数据进行反序列化。

关于错误检查则是一个开放性的问题。我们现在再来看下`operator>>`实现：

```c++
istream& operator>>(istream &is, city &c)
{
    is >> ws;
    getline(is, c.name);
    is >> c.population >> c.latitude >> c.longitude;
    return is;
}
```

我们读取了很多不同的东西。读取数据发生了错误，下一个应该怎么办？这是不是意味着我们有可能读取到错误的数据？不会的，这不可能发生。即便是其中一个元素没有被输入流进行解析，那么输入流对象则会置于错误的状态，并且拒绝对剩下的输入进行解析。这样就意味着，如果`c.population`或`c.latitude`没有被解析出来，那么对应的输入数据将会被丢弃，并且我们可以看到反序列了一半的`city`对象。

站在调用者的角度，我们需要注意这句`if(input_stream >> city_object)`。这也就表面流表达式将会被隐式转换成一个布尔值。当其返回false时，输入流对象则处于错误状态。如果出现错误，就需要采取相应的措施对流进行重置。

本节中没有使用`if`判断，因为我们让`  std::istream_iterator<city>`进行反序列化。`operator++`在迭代器的实现中，会在解析时对其状态进行检查。当遇到错误时，其将会停止之后的所有迭代。当前迭代器与`end`迭代器比较返回true时，将终止`copy`算法的执行。如此，我们的代码就很安全了。