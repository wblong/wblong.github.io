title: 条件变量(condition_variable)
author: Nico
tags:
  - C++11
categories:
  - C++
date: 2019-05-25 18:08:00
---
#### std::condition_variable

`std::condition_variable`是条件变量,当`std::condition_variable`对象的某个`wait `函数被调用的时候，它使用 `std::unique_lock`(通过 `std::mutex`) 来锁住当前线程。当前线程会一直被阻塞，直到另外一个线程在相同的 `std::condition_variable` 对象上调用了` notification` 函数来唤醒当前线程。

```
#include <iostream>                // std::cout
#include <thread>                // std::thread
#include <mutex>                // std::mutex, std::unique_lock
#include <condition_variable>    // std::condition_variable

std::mutex mtx; // 全局互斥锁.
std::condition_variable cv; // 全局条件变量.
bool ready = false; // 全局标志位.

void do_print_id(int id)
{
	std::unique_lock <std::mutex> lck(mtx);
	while (!ready) // 如果标志位不为 true, 则等待...
		cv.wait(lck); // 当前线程被阻塞, 当全局标志位变为 true 之后,
	// 线程被唤醒, 继续往下执行打印线程编号id.
	std::cout << "thread " << id << '\n';
}

void go()
{
	std::unique_lock <std::mutex> lck(mtx);
	ready = true; // 设置全局标志位为 true.
	cv.notify_all(); // 唤醒所有线程.
}

int main()
{
	std::thread threads[10];
	// spawn 10 threads:
	for (int i = 0; i < 10; ++i)
		threads[i] = std::thread(do_print_id, i);
	std::cout << "10 threads ready to race...\n";
	go(); // go!
	for (auto & th : threads)
		th.join();
	return 0;
}
```

只有当 pred 条件为false 时调用 wait() 才会阻塞当前线程，并且在收到其他线程的通知后只有当 pred 为 true 时才会被解除阻塞。
```
#include <iostream>                // std::cout
#include <thread>                // std::thread, std::this_thread::yield
#include <mutex>                // std::mutex, std::unique_lock
#include <condition_variable>    // std::condition_variable

std::mutex mtx;
std::condition_variable cv;

int cargo = 0;
bool shipment_available()
{
	return cargo != 0;
}

// 消费者线程.
void consume(int n)
{
	for (int i = 0; i < n; ++i) {
		std::unique_lock <std::mutex> lck(mtx);
		cv.wait(lck, shipment_available);
		std::cout << cargo << '\n';
		cargo = 0;
	}
}

int main()
{
	std::thread consumer_thread(consume, 10); // 消费者线程.

	// 主线程为生产者线程, 生产 10 个物品.
	for (int i = 0; i < 10; ++i) {
		while (shipment_available())
			std::this_thread::yield();//让出CPU片
		std::unique_lock <std::mutex> lck(mtx);
		cargo = i + 1;
		cv.notify_one();
	}

	consumer_thread.join();

	return 0;
}
```

与`std::condition_variable::wait()` 类似，不过 `wait_for`可以指定一个时间段，在当前线程收到通知或者指定的时间 rel_time 超时之前，该线程都会处于阻塞状态。而一旦超时或者收到了其他线程的通知，`wait_for`返回，剩下的处理步骤和` wait()`类似。
```
#include<iostream>
#include <thread>             // std::thread
#include <chrono>             // std::chrono::seconds
#include <mutex>              // std::mutex, std::unique_lock
#include <condition_variable> // std::condition_variable, std::cv_status

std::condition_variable cv;

int value;

void do_read_value()
{
	std::cin >> value;
	cv.notify_one();
}

int main()
{
	std::cout << "Please, enter an integer (I'll be printing dots): \n";
	std::thread th(do_read_value);

	std::mutex mtx;
	std::unique_lock<std::mutex> lck(mtx);
	while (cv.wait_for(lck, std::chrono::seconds(1)) == std::cv_status::timeout) {
		std::cout << '.';
		std::cout.flush();
	}

	std::cout << "You entered: " << value << '\n';

	th.join();
	return 0;
}
```
#### std::condition_variable_any
与 `std::condition_variable`类似，只不过`std::condition_variable_any`的 wait 函数可以接受任何 lockable参数，而 `std::condition_variable`只能接受 `std::unique_lock`类型的参数
