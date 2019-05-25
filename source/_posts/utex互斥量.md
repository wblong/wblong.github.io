title: mutex互斥量与锁
author: Nico
tags:
  - C++11
categories:
  - C++
  - ''
date: 2019-05-25 17:21:00
---
`mutex` 互斥对象，保护代码片段同一时刻只被一个线程访问。
`mutex` 可能在程序异常时无法释放锁。可以结合`unique_lock`和`std::lock_guard`使用来自动加锁和释放锁。

```
#include<iostream>
#include<mutex>
#include<thread>
std::mutex mtx;

static void print_block(int n, char c) {
 mtx.lock();
 for (int i = 0; i < n; ++i) {
  std::cout << c;
 }
 std::cout << "\n";
 mtx.unlock();
}

int test_mutex_1() {
 std::thread th1(print_block, 50, '*');
 std::thread th2(print_block, 50, '&');
 th1.join();
 th2.join();
 return 0;
}

static void print_thread_id(int id) {
 mtx.lock();
 std::cout << "thread #" << id << "\n";
 mtx.unlock();
}

int test_mutex_2() {
 std::thread threads[10];
 for (int i = 0; i < 10; ++i) {
  threads[i] =  std::thread(print_thread_id, i + 1);
 }
 for (auto& th : threads) {
  th.join();
 }
 return 0;
}

volatile int counter(0);
static void attempt_10k_increases() {
 for (int i = 0; i < 10000; ++i) {
  
  if (mtx.try_lock()) {
   ++counter;
   mtx.unlock();
  }
  /*mtx.lock();
  ++counter;
  mtx.unlock();*/
 }
}
int test_mutex_3()
{
 std::thread threads[10];
 // spawn 10 threads:
 for (int i = 0; i < 10; ++i)
  threads[i] = std::thread(attempt_10k_increases);
 for (auto& th : threads) th.join();
 std::cout << counter << " successful increases of the counter.\n";
 return 0;
}

int main(int *argc,char*agv[]) {
 //test_mutex_1();
 //test_mutex_2();
 test_mutex_3();
 return 0;
}

```
`std::lock_guard`简单锁，在构造函数中进行加锁，析构函数中进行解锁。

```
#include<mutex>
#include<thread>
#include<iostream>
#include<vector>
#include<algorithm>

std::mutex mtx;

void add(int &num, int&sum) {
	while (true)
	{
		std::lock_guard<std::mutex>lk(mtx);
		if (num < 100) {
			++num;
			sum += num;

		}
		else {
			break;
		}
	}
}

int main() {
	int sum = 0, num = 0;
	std::vector<std::thread>ver_ths;
	for (int i = 0; i < 20; ++i) {
		std::thread th = std::thread(add, std::ref(num), std::ref(sum));
		// 较之push_back 只调用构造函数，没有移动构造函数，也没有拷贝构造函数
		ver_ths.emplace_back(std::move(th));
	}
	//使用mem_fun来取指针并绑定到std::thread的成员函数
	std::for_each(ver_ths.begin(), ver_ths.end(), std::mem_fn(&std::thread::join));
	std::cout << sum << std::endl;
	return 0;
}
```

`std::unique_lock`延迟锁定、锁定的有时限尝试、递归锁定、所有权转移和与条件变量一同使用。
`unique_lock`比`lock_guard`使用更加灵活，功能更加强大。
使用`unique_lock`需要付出更多的时间、性能成本。

```
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::unique_lock
#include <vector>

std::mutex mtx;           // mutex for critical section
std::once_flag flag;

void print_block(int n, char c) {
	//unique_lock有多组构造函数, 这里std::defer_lock不设置锁状态
	std::unique_lock<std::mutex> my_lock(mtx, std::defer_lock);
	//尝试加锁, 如果加锁成功则执行
	//(适合定时执行一个job的场景, 一个线程执行就可以, 可以用更新时间戳辅助)
	if (my_lock.try_lock()) {
		for (int i = 0; i < n; ++i)
			std::cout << c;
		std::cout << '\n';
	}
}

void run_one(int &n) {
	std::call_once(flag, [&n] {n = n + 1; }); //只执行一次, 适合延迟加载; 多线程static变量情况
}

int main() {
	std::vector<std::thread> ver;
	int num = 0;
	for (auto i = 0; i < 10; ++i) {
		ver.emplace_back(print_block, 50, '*');
		ver.emplace_back(run_one, std::ref(num));
	}

	for (auto &t : ver) {
		t.join();
	}
	std::cout << num << std::endl;
	return 0;
}
```