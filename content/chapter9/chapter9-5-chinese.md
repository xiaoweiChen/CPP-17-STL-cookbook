# 避免死锁——std::scoped_lock

如果在路上发生了死锁，就会像下图一样：

![](../../images/chapter9/9-5-1.png)

为了让交通顺畅，可能需要一个大型起重机，将路中间的一辆车挪到其他地方去。如果找不到起重机，那么我们就希望这些司机们能互相配合。当几个司机愿意将车往后退，留给空间给其他车通行，那么每辆车就不会停在原地了。

多线程编程中，开发者肯定需要避免这种情况的发生。不过，程序比较复杂的情况下，这种情况其实很容易发生。

本节中，我们将会故意的创造一个死锁的情况。然后，在相同资源的情况下，如何创造出一个死锁的情形。再使用C++17中，STL的`std::scoped_lock`如何避免死锁的发生。

## How to do it...

本节中有两对函数要在并发的线程中执行，并且有两个互斥量。其中一对制造死锁，另一对解决死锁。主函数中，我们将使用这两个互斥量：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <thread>
   #include <mutex>
   
   using namespace std;
   using namespace chrono_literals;
   ```

2. 实例化两个互斥量对象，制造死锁：

   ```c++
   mutex mut_a;
   mutex mut_b;
   ```

3. 为了使用两个互斥量制造死锁，我们需要有两个函数。其中一个函数试图对互斥量`A`进行上锁，然后对互斥量B进行上锁，而另一个函数则试图使用相反的方式运行。让两个函数在等待锁时进行休眠，我们确定这段代码永远处于一个死锁的状态。(这就达到了我们演示的目的。当我们重复运行程序，那么程序在没有任何休眠代码的同时，可能会有成功运行的情况。)需要注意的是，这里我们没有使用`\n`字符作为换行符，我们使用的是`endl`。`endl`会输出一个换行符，同时也会对`cout`的流缓冲区进行刷新，所以我们可以确保打印信息不会有延迟或同时出现：

   ```c++
   static void deadlock_func_1()
   {
       cout << "bad f1 acquiring mutex A..." << endl;
       
       lock_guard<mutex> la {mut_a};
       
       this_thread::sleep_for(100ms);
       
       cout << "bad f1 acquiring mutex B..." << endl;
       
       lock_guard<mutex> lb {mut_b};
       
       cout << "bad f1 got both mutexes." << endl;
   }
   ```

4. `deadlock_func_2`和`deadlock_func_1`看起来一样，就是`A`和`B`的顺序相反：

   ```c++
   static void deadlock_func_2()
   {
       cout << "bad f2 acquiring mutex B..." << endl;
       
       lock_guard<mutex> lb {mut_b};
       
       this_thread::sleep_for(100ms);
       
       cout << "bad f2 acquiring mutex A..." << endl;
       
       lock_guard<mutex> la {mut_a};
       
       cout << "bad f2 got both mutexes." << endl;
   }
   ```

5. 现在我们将完成与上面函数相比，两个无死锁版本的函数。它们使用了`scoped_lock`，其会作为构造函数参数的所有互斥量进行上锁。其析构函数会进行解锁操作。锁定这些互斥量时，其内部应用了避免死锁的策略。这里需要注意的是，两个函数还是对`A`和`B`互斥量进行操作，并且顺序相反：

   ```c++
   static void sane_func_1()
   {
   	scoped_lock l {mut_a, mut_b};
   	
       cout << "sane f1 got both mutexes." << endl;
   }
   
   static void sane_func_2()
   {
   	scoped_lock l {mut_b, mut_a};
   	
       cout << "sane f2 got both mutexes." << endl;
   }
   ```

6. 主函数中观察这两种情况。首先，我们使用不会死锁的函数：

   ```c++
   int main()
   {
       {
           thread t1 {sane_func_1};
           thread t2 {sane_func_2};
           
           t1.join();
           t2.join();
       }
   ```

7. 然后，调用制造死锁的函数：

   ```c++
       {
           thread t1 {deadlock_func_1};
           thread t2 {deadlock_func_2};
           
           t1.join();
           t2.join();
       }
   }
   ```

8. 编译并运行程序，就能得到如下的输出。前两行为无死锁情况下，两个函数的打印结果。接下来的两个函数则产生死锁。因为我们能看到f1函数始终是在等待互斥量B，而f2则在等待互斥量A。两个函数都没做成功的对两个互斥量上锁。我们可以让这个程序持续运行，不管时间是多久，结果都不会变化。程序只能从外部进行杀死，这里我们使用`Ctrl + C`的组合键，将程序终止：

   ```c++
   $ ./avoid_deadlock
   sane f1 got both mutexes
   sane f2 got both mutexes
   bad f2 acquiring mutex B...
   bad f1 acquiring mutex A...
   bad f1 acquiring mutex B...
   bad f2 acquiring mutex A...
   ```

## How it works...

例子中，我们故意制造了死锁，我们也了解了这样一种情况发生的有多快。在一个很大的项目中，多线程开发者在编写代码的时候，都会共享一些互斥量用于保护资源，所有开发者都需要遵循同一种加锁和解锁的顺序。这种策略或规则是很容易遵守的，不过也是很容易遗忘的。另一个问题则是*锁序倒置*。

`scoped_lock`对于这种情况很有帮助。其实在C++17中添加，其工作原理与`lock_guard`和`unique_lock`一样：其构造函数会进行上锁操作，并且析构函数会对互斥量进行解锁操作。`scoped_lock`特别之处是，可以指定多个互斥量。

`scoped_lock`使用`std::lock`函数，其会调用一个特殊的算法对所提供的互斥量调用`try_lock`函数，这是为了避免死锁。因此，在加锁与解锁的顺序相同的情况下，使用`scoped_lock`或对同一组锁调用`std::lock`都是非常安全的。