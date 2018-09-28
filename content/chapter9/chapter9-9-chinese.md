# 实现生产者/消费者模型——std::condition_variable

本节中，我们将使用多线程实现一个经典的生产者/消费者模型。其过程就是一个生产者线程将商品放到队列中，然后另一个消费者线程对这个商品进行消费。如果不需要生产，生产者线程休眠。如果队列中没有商品能够消费，消费者休眠。

这里两个线程都需要对同一个队列进行修改，所以我们需要一个互斥量来保护这个队列。

需要考虑的事情是：如果队列中没有商品了，那么消费者做什么呢？需要每秒对队列进行检查，直到看到新的商品吗？当然，我们可以通过生产者触发一些事件叫醒消费者，这样消费者就能在第一时间获取到新的商品。

C++11中提供了一个很不错的数据结构` std::condition_variable`，其很适合处理这样的情况。本节中，我们实现一个简单的生产者/消费者应用，来对这个数据结构进行使用。

## How to do it...

我们将实现一个单生产者/消费者程序，每个角色都有自己的线程：

1. 包含必要的头文件，并且声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <queue>
   #include <tuple>
   #include <condition_variable>
   #include <thread>
   
   using namespace std;
   using namespace chrono_literals;
   ```

2. 队列进行实例化，并且队列`q`里只放简单的数字。生产者将商品放入队列中，消费者将商品从队列中取出。为了进行同步，我们需要一个互斥量。这就需要我们对`condition_variable`进行实例化，其变量名为`cv`。`finished`变量将会告诉生产者，无需在继续生产：

   ```c++
   queue<size_t> q;
   mutex mut;
   condition_variable cv;
   bool finished {false};
   ```

3. 首先，我们来实现一个生产者函数。其能接受一个参数`items`，其会限制生产者生产的最大数量。一个简单的循环中，我们将会隔100毫秒生产一个商品，这个耗时就是在模拟生产的复杂性。然后，我们会对队列的互斥量进行上锁，并同步的对队列进行访问。成功的生产后，将商品加入队列时，我们需要调用`cv.notify_all()`，函数会叫醒所有消费线程。我们将在后面看到消费者那边是如何工作的：

   ```c++
   static void producer(size_t items) {
       for (size_t i {0}; i < items; ++i) {
           this_thread::sleep_for(100ms);
           {
               lock_guard<mutex> lk {mut};
               q.push(i);
           }
           cv.notify_all();
       }
   ```

4. 生产完所有商品后，我们会将互斥量再度上锁，因为需要对`finished`位进行设置。然后，再次调用`cv.notify_all()`：

   ```c++
       {
           lock_guard<mutex> lk {mut};
           finished = true;
       }
       cv.notify_all();
   }
   ```

5.  现在来实现消费者函数。因为只是对队列上的数值进行消费，直到消费完所有的数值，所以这个函数不需要参数。当`finished`未被设置时，循环会持续执行，并且会对保护队列的互斥量进行上锁，将对队列和`finished`标识同时进行保护。当互斥量上锁，则锁就会调用`cv.wait`，并以Lambda表达式为参数。这个Lambda表达式其实就是个条件谓词，如果生产者线程还在继续工作，并且还有商品在队列上，消费者线程就不能停下来：

   ```c++
   static void consumer() {
       while (!finished) {
           unique_lock<mutex> l {mut};
           
           cv.wait(l, [] { return !q.empty() || finished; });
   ```

6. `  cv.wait`的调用会对锁进行解锁，并且会等到给予的条件达成时才会继续运行。然后，其会再次对互斥量上锁，并对队列上的商品进行消费，直到队列为空。如果生成者还在继续生成，那么这个循环可能会一直进行下去。否则，当`finished`被设置时，循环将会终止，这也就表示生产者不会再进行生产：

   ```c++
           while (!q.empty()) {
               cout << "Got " << q.front()
               	<< " from queue.\n";
               q.pop();
           }
       }
   }
   ```

7. 主函数中，我们让生产者生产10个商品。然后，我们就等待程序的结束：

   ```c++
   int main() {
       thread t1 {producer, 10};
       thread t2 {consumer};
       t1.join();
       t2.join();
       cout << "finished!\n";
   }
   ```

8. 编译并运行程序，我们将会得到下面的输出。当程序在运行阶段时，我们将看到每一行，差不多隔100毫秒打印出来，因为生产者需要时间进行生产：

   ```c++
   $ ./producer_consumer
   Got 0 from queue.
   Got 1 from queue.
   Got 2 from queue.
   Got 3 from queue.
   Got 4 from queue.
   Got 5 from queue.
   Got 6 from queue.
   Got 7 from queue.
   Got 8 from queue.
   Got 9 from queue.
   finished!
   ```

## How it works...

本节中，我们只启动了两个线程。第一个线程会生产一些商品，并放到队列中。另一个则是从队列中取走商品。当其中一个线程需要对队列进行访问时，其否需要对公共互斥量`mut`进行上锁，这样才能对队列进行访问。这样，我们就能保证两个线程不能再同时对队列进行操作。

除了队列和互斥量，我们还声明了2个变量，其也会对生产者和消费者有所影响：

```c++
queue<size_t> q;
mutex mut;
condition_variable cv;
bool finished {false};
```

`finished`变量很容易解释。当其设置为true时，生产者则会对固定数量的产品进行生产。当消费者看到这个值为true的时候，其就要将队列中的商品全部消费完。但是` condition_variable cv`代表了什么呢？我们在两个不同上下文中使用`cv`。其中一个上下文则会去等待一个特定的条件，并且另一个会达成对应的条件。

消费者这边将会等待一个特殊的条件。消费者线程会在对互斥量`mut`使用`unique_lock`上锁后，进行阻塞循环。然后，会调用`cv.wait`：

```c++
while (!finished) {
    unique_lock<mutex> l {mut};
    
    cv.wait(l, [] { return !q.empty() || finished; });
    
    while (!q.empty()) {
    	// consume
    }
}
```

我们写了下面一段代码，这上下来两段代码看起来是等价的。我们会在后面了解到，这两段代码真正的区别到底在哪里：

```c++
while (!finished) {
    unique_lock<mutex> l {mut};
    
    while (q.empty() && !finished) {
        l.unlock();
        l.lock();
    }
   
    while (!q.empty()) {
    	// consume
    }
}
```

这就意味着，我们先要进行上锁，然后对我们的应对方案进行检查：

1. 还有商品能够消费吗？有的话，继续持有锁，消费，释放锁，结束。
2. 如果没有商品可以消费，但是生产者依旧存在，释放互斥锁，也就是给生产者一个机会向队列中添加商品。然后，尝试再对现状进行检查，如果现状有变，则跳转到1方案中。

`cv.wait`为什么与`while(q.empty() && ... )`不等价呢？因为在`wait`不需要再循环中持续的进行上锁和解锁的循环。如果生产者线程处于未激活状态时，这就会导致互斥量持续的被上锁和解锁，这样的操作是没有意义的，而且还会耗费掉不必要的CPU周期。

` cv.wait(lock, predicate)`将会等到` predicate()`返回true时，结束等待。不过其不会对`lock`持续的进行解锁与上锁的操作。为了将使用`wait`阻塞的线程唤醒，我们就需要使用`condition_variable` 对象，另一个线程会对同一个对象调用`notify_one()`或`notify_all()`。等待中的线程将从休眠中醒来，并检查`predicate()`条件是否成立。

`wait`还有一个很好的地方在于，如果出现了伪唤醒操作，那么线程则会再次进行休眠状态。这也就是当我们发出了过多的叫醒信号时，其不会对程序流有任何影响(但是会影响到性能)。

生产者端，在向队列输出商品后，调用` cv.notify_all()`，并且在生产最后一个商品时，将`finished`设置为true，这就等于引导了消费者进行消费。