# 实现生产者/消费者模型——std::condition_variable

让我们再来回顾一下生产者/消费者问题，这要比上一节的问题更加复杂。我们创建了多个生成这和多个消费者。并且，我们定义的队列没有限制上限。

当队列中没有商品的时候，消费者会处于等待状态，并且当队列中没有空间可以盛放商品时，生产者也会处于等待状态。

我们将会使用多个`std::condition_variable`对象来解决这个问题，不过使用的方式与上节有些不同。

## How to do it...

本节中，我们将实现一个类似于上节的程序，不过，这次我们有多个生产者和消费者：

1. 包含必要的头文件，并声明所使用的命名空间：

   ```c++
   #include <iostream>
   #include <iomanip>
   #include <sstream>
   #include <vector>
   #include <queue>
   #include <thread>
   #include <mutex>
   #include <condition_variable>
   #include <chrono>
   
   using namespace std;
   using namespace chrono_literals;
   ```

2. 接下来从本章的其他小节中拿过来一个同步打印的辅助类型，因为其能帮助我们在大量并发时进行打印：

   ```c++
   struct pcout : public stringstream {
       static inline mutex cout_mutex;
       ~pcout() {
           lock_guard<mutex> l {cout_mutex};
           cout << rdbuf();
       }
   };
   ```

3. 所有生产者都会将值写入到同一个队列中，并且所有消费者也会从这个队列中获取值。对于这个队列，我们需要使用一个互斥量对队列进行保护，并设置一个标识，其会告诉我们生产者是否已经停止生产：

   ```c++
   queue<size_t> q;
   mutex q_mutex;
   bool production_stopped {false};
   ```

4. 我们将在程序中使用两个`condition_variable`对象。在单生产者/消费者的情况下，我们只需要一个`condition_variable`告诉我们队列上面摆放了新商品。在本节的例子中，我们将来处理更加复杂的情况。我们需要生产者持续生产，以保证队列上一直有可消费的商品存在。如果商品囤积到一定程度，则生产者休息。` go_consume`变脸高就用来提醒消费者消费的，而`go_produce`则是用来提醒生产者进行生产的：

   ```c++
   condition_variable go_produce;
   condition_variable go_consume;
   ```

5. 生产者函数能够接受一个生产者ID，所要生产的商品数量，以及囤积商品值的上限。然后，生产者就会进入循环生产阶段。这里，首先其会对队列的互斥量进行上锁，然后在通过调用` go_produce.wait`对互斥量进行解锁。队列上的商品数量未达到囤积阈值时，满足等待条件：

   ```c++
   static void producer(size_t id, size_t items, size_t stock)
   {
       for (size_t i = 0; i < items; ++i) {
           unique_lock<mutex> lock(q_mutex);
           go_produce.wait(lock,
           	[&] { return q.size() < stock; });
   ```

6. 生产者开始生产后，就会有商品放入队列中。队列商品的表达式为`id * 100 + i`。因为百位和线程id强相关，这样我们就能了解到哪些商品是哪些生产者生产的。我们也能将生产事件打印到终端上。格式看起来可能有奇怪，不过其会与消费者打印输出相对其：

   ```c++
           q.push(id * 100 + i);
           
   		pcout{} << " Producer " << id << " --> item "
           		<< setw(3) << q.back() << '\n';
   ```

7. 生茶之后，我们叫醒沉睡终中的消费者。每个睡眠周期为90毫秒，这用来模拟生产者生产商品的时间：

   ```c++
           go_consume.notify_all();
           this_thread::sleep_for(90ms);
       }
   
       pcout{} << "EXIT: Producer " << id << '\n';
   }
   ```

8. 现在来时先消费者函数，其只接受一个消费者ID作为参数。当生产者停止生产，或是队列上没有商品，消费者就会继续等待。当队列上没有商品，但是生产者正在生产，那么可以肯定的是，队列上将会有新商品的产生：

   ```c++
   static void consumer(size_t id)
   {
       while (!production_stopped || !q.empty()) {
       	unique_lock<mutex> lock(q_mutex);
   ```

9. 在对队列的互斥量上锁之后，我们将会在等待`go_consume`事件变量时对互斥量进行解锁。Lambda表达式说明，当队列中有商品的时候我们就会对其进行获取。第二个参数`1s`说明，我们并不想等太久。如果等待时间超过1秒，我们将不会等待。当谓词条件达成，则`wait_for`函数返回；否则就会因为超时而放弃等待。如果队列中有新商品，我们将会获取这个商品，并进行相应的打印：

   ```c++
   		if (go_consume.wait_for(lock, 1s,
   				[] { return !q.empty(); })) {
   			pcout{} << " item "
   					<< setw(3) << q.front()
   					<< " --> Consumer "
   					<< id << '\n';
   			q.pop();
   ```

10. 商品被消费之后，我们将会提醒生产者，并进入130毫秒的休眠状态，这个时间用来模拟消费时间：

    ```c++
                go_produce.notify_all();
                this_thread::sleep_for(130ms);
            }
        }
    
        pcout{} << "EXIT: Producer " << id << '\n';
    }
    ```

11. 主函数中，我们对工作线程和消费线程各自创建一个vector:

    ```c++
    int main()
    {
        vector<thread> workers;
        vector<thread> consumers;
    ```

12. 然后，我们创建3个生产者和5个消费者：

    ```c++
        for (size_t i = 0; i < 3; ++i) {
        	workers.emplace_back(producer, i, 15, 5);
        }
    
        for (size_t i = 0; i < 5; ++i) {
        	consumers.emplace_back(consumer, i);
        }
    ```

13. 我们会先让生产者线程终止。然后返回，并对`production_stopped`标识进行设置，这将会让消费者线程同时停止。然后，我们要将这些线程进行回收，然后退出程序：

    ```c++
        for (auto &t : workers) { t.join(); }
        production_stopped = true;
        for (auto &t : consumers) { t.join(); }
    }
    ```

14. 编译并运行程序，我们将获得如下的输出。这个输出特别长，我们进行了截断。我们能看到生产者偶尔会休息一下，并且消费者会消费掉对应的商品，直到再次生产。





## How it works...

