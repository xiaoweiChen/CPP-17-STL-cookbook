# 实现多生产者/多消费者模型——std::condition_variable

让我们再来回顾一下生产者/消费者问题，这要比上一节的问题更加复杂。我们创建了多个生成这和多个消费者。并且，我们定义的队列没有限制上限。

当队列中没有商品时，消费者会处于等待状态，而当队列中没有空间可以盛放商品时，生产者也会处于等待状态。

我们将会使用多个`std::condition_variable`对象来解决这个问题，不过使用的方式与上节有些不同。

## How to do it...

本节中，我们将实现一个类似于上节的程序，这次我们有多个生产者和消费者：

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

4. 我们将在程序中使用两个`condition_variable`对象。单生产者/消费者的情况下，只需要一个`condition_variable`告诉我们队列上面摆放了新商品。本节的例子中，我们将来处理更加复杂的情况。我们需要生产者持续生产，以保证队列上一直有可消费的商品存在。如果商品囤积到一定程度，则生产者休息。` go_consume`变量就用来提醒消费者消费的，而`go_produce`则是用来提醒生产者进行生产的：

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

6. 生产者开始生产后，就会有商品放入队列中。队列商品的表达式为`id * 100 + i`。因为百位和线程id强相关，这样我们就能了解到哪些商品是哪些生产者生产的。我们也能将生产事件打印到终端上。格式看起来可能有些奇怪，不过其会与消费者打印输出对齐：

   ```c++
           q.push(id * 100 + i);
           
   		pcout{} << " Producer " << id << " --> item "
           		<< setw(3) << q.back() << '\n';
   ```

7. 生产商品之后，我们叫醒沉睡中的消费者。每个睡眠周期为90毫秒，这用来模拟生产者生产商品的时间：

   ```c++
           go_consume.notify_all();
           this_thread::sleep_for(90ms);
       }
   
       pcout{} << "EXIT: Producer " << id << '\n';
   }
   ```

8. 现在来实现消费者函数，其只接受一个消费者ID作为参数。当生产者停止生产，或是队列上没有商品，消费者就会继续等待。队列上没有商品时，生产者还在生产的话，那么可以肯定的是，队列上肯定会有新商品的产生：

   ```c++
   static void consumer(size_t id)
   {
       while (!production_stopped || !q.empty()) {
       	unique_lock<mutex> lock(q_mutex);
   ```

9. 对队列的互斥量上锁之后，我们将会在等待`go_consume`事件变量时对互斥量进行解锁。Lambda表达式表明，当队列中有商品的时候我们就会对其进行获取。第二个参数`1s`说明，我们并不想等太久。如果等待时间超过1秒，我们将不会等待。当谓词条件达成，则`wait_for`函数返回；否则就会因为超时而放弃等待。如果队列中有新商品，我们将会获取这个商品，并进行相应的打印：

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

11. 主函数中，我们对工作线程和消费线程各自创建一个`vector`:

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

14. 编译并运行程序，我们将获得如下的输出。输出特别长，我们进行了截断。我们能看到生产者偶尔会休息一下，并且消费者会消费掉对应的商品，直到再次生产。若是将生产者/消费者的休眠时间进行修改，则会得到完全不一样的结果：

    ```c++
    $ ./multi_producer_consumer
    Producer 0 --> item 0
    Producer 1 --> item 100
    item 0 --> Consumer 0
    Producer 2 --> item 200
    item 100 --> Consumer 1
    item 200 --> Consumer 2
    Producer 0 --> item 1
    Producer 1 --> item 101
    item 1 --> Consumer 0
    ...
    Producer 0 --> item14
    EXIT: Producer 0
    Producer 1 --> item 114
    EXIT: Producer 1
    item14 --> Consumer 0
    Producer 2 --> item 214
    EXIT: Producer 2
    item 114 --> Consumer 1
    item 214 --> Consumer 2
    EXIT: Consumer 2
    EXIT: Consumer 3
    EXIT: Consumer 4
    EXIT: Consumer 0
    EXIT: Consumer 1
    ```

## How it works...

这节可以作为之前章节的扩展。与单生产者和消费者不同，我们实现了M个生产者和N个消费者之间的同步。因此，程序中不是消费者因为队列中没有商品而等待，就是因为队列中囤积了太多商品让生产者等待。

当有多个消费者等待同一个队列中出现新的商品时，程序的模式就又和单生产者/消费者工作的模式相同了。当有一个线程对保护队列的互斥量上锁时，然后对货物进行添加或减少，这样代码就是安全的。这样的话，无论有多少线程在同时等待这个锁，对于我们来说都无所谓。生产者也同理，其中最重要的就是，队列不允许两个及两个以上的线程进行访问。

比单生产者/消费者原理复杂的原因在于，当商品的数量在队列中囤积到一定程度，我们将会让生产者线程停止。为了迎合这个需求，我们使用两个不同的`condition_variable`：

1. `go_produce`表示队列没有被填满，并且生产者会继续生产，而后将商品放置在队列中。
2. `go_consume`表示队列已经填满，消费者可以继续消费。

这样，生产者会将队列用货物填满，并且`go_consume`会用如下代码，提醒消费者线程：

```c++
if (go_consume.wait_for(lock, 1s, [] { return !q.empty(); })) {
	// got the event without timeout
}
```

生产者也会进行等待，直到可以再次生产：

```c++
go_produce.wait(lock, [&] { return q.size() < stock; });
```

还有一个细节就是我们不会让消费者线程等太久。在对`go_consume.wait_for`的调用中，我们添加了超时参数，并且设置为1秒。这对于消费者来说是一种退出机制：当队列为空的状态持续多于1秒，那么就可能没有生产者在工作。

这个处理起来很简单，代码会尽可能让队列中商品的数量达到阈值的上限。更复杂的程序中，当商品的数量为阈值上限的一半时，消费者线程会对生产者线程进行提醒。这样生产者就会在队列为空前继续生产。

` condition_variable`帮助我们完美的解决了一个问题：当一个消费者触发了`go_produce`的提醒，那么将会有很多生产者竞争的去生产下一个商品。如果只需要生产一个商品，那么只需要一个生产者就好。当`go_produce`被触发时，所有生产者都争相生产这一个商品，我们将会看到的情况就是商品在队列中的数量超过了阈值的上限。

我们试想一下这种情况，我们有`(max - 1)`个商品在队列中，并且想在要一个商品将队列填满。不论是一个消费者线程调用了`go_produce.notify_one()`(只叫醒一个等待线程)或` go_produce.notify_all()`(叫醒所有等待的线程)，都需要保证只有一个生产者线程调用了`go_produce.wait`，因为对于其他生成线程来说，一旦互斥锁解锁，那么`q.size() < stock`(stock货物阈值上限)的条件将不复存在。