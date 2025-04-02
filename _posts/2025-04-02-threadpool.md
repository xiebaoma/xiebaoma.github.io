---
title: "Threadpool demo"
subtitle: "C++ threadpool implementation for Linux"
layout: post
author: "细胞膜"
header-style: text
date:       2025-04-02 12:01:00
hidden: true
tags:
  - C++
  - project
---

# Threadpool demo

This is a C++ threadpool implementation for Linux that demonstrates how to efficiently manage and distribute tasks across multiple threads.

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <future>
#include <atomic>

class ThreadPool {
public:
    // Constructor: initializes the thread pool with specified number of worker threads
    ThreadPool(size_t numThreads) : stop(false) {
        for (size_t i = 0; i < numThreads; ++i) {
            workers.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    
                    {
                        std::unique_lock<std::mutex> lock(queueMutex);
                        condition.wait(lock, [this] { 
                            return stop || !tasks.empty(); 
                        });
                        
                        if (stop && tasks.empty()) {
                            return;
                        }
                        
                        task = std::move(tasks.front());
                        tasks.pop();
                    }
                    
                    task();
                }
            });
        }
    }
    
    // Destructor: stops all threads and joins them
    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queueMutex);
            stop = true;
        }
        
        condition.notify_all();
        
        for (std::thread &worker : workers) {
            if (worker.joinable()) {
                worker.join();
            }
        }
    }
    
    // Deleted copy constructor and assignment operator
    ThreadPool(const ThreadPool &) = delete;
    ThreadPool &operator=(const ThreadPool &) = delete;
    
    // Enqueue a task and get a future to its result
    template<class F, class... Args>
    auto enqueue(F&& f, Args&&... args) -> std::future<typename std::result_of<F(Args...)>::type> {
        using return_type = typename std::result_of<F(Args...)>::type;
        
        auto task = std::make_shared<std::packaged_task<return_type()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        
        std::future<return_type> result = task->get_future();
        
        {
            std::unique_lock<std::mutex> lock(queueMutex);
            
            if (stop) {
                throw std::runtime_error("enqueue on stopped ThreadPool");
            }
            
            tasks.emplace([task]() { (*task)(); });
        }
        
        condition.notify_one();
        return result;
    }
    
    // Get the number of worker threads
    size_t size() const {
        return workers.size();
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    
    std::mutex queueMutex;
    std::condition_variable condition;
    bool stop;
};

// Example usage
int main() {
    // Create a thread pool with the number of hardware threads available
    ThreadPool pool(std::thread::hardware_concurrency());
    std::cout << "Thread pool created with " << pool.size() << " threads\n";
    
    // Vector to store futures
    std::vector<std::future<int>> results;
    
    // Example 1: Basic task submission
    for (int i = 0; i < 20; ++i) {
        auto result = pool.enqueue([i] {
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
            printf("Task %d executed by thread %ld\n", i, pthread_self());
            return i * i;
        });
        results.push_back(std::move(result));
    }
    
    // Wait for and print results
    for (auto& result : results) {
        std::cout << "Result: " << result.get() << std::endl;
    }
    
    // Example 2: Calculating sum using multiple threads
    std::atomic<long> sum(0);
    std::vector<std::future<void>> sumTasks;
    
    const int numItems = 1000000;
    const int batchSize = 10000;
    
    for (int i = 0; i < numItems; i += batchSize) {
        auto task = pool.enqueue([&sum, i, batchSize] {
            long localSum = 0;
            int end = std::min(i + batchSize, numItems);
            for (int j = i; j < end; ++j) {
                localSum += j;
            }
            sum += localSum;
        });
        sumTasks.push_back(std::move(task));
    }
    
    // Wait for all sum tasks to complete
    for (auto& task : sumTasks) {
        task.get();
    }
    
    std::cout << "Sum of numbers from 0 to " << (numItems - 1) << ": " << sum << std::endl;
    
    // The ThreadPool destructor will automatically join all threads
    std::cout << "Exiting program, thread pool will be destroyed\n";
    
    return 0;
}
```

To compile and run this threadpool implementation on Linux:

```bash
g++ -std=c++11 -pthread threadpool.cpp -o threadpool
./threadpool
```

This implementation includes:

1. A robust `ThreadPool` class that:
   - Creates a pool of worker threads
   - Manages a queue of tasks
   - Provides a clean way to enqueue tasks and get their results via futures
   - Properly handles shutdown and cleanup
2. Two practical examples:
   - A simple task that shows which thread executes each task
   - A parallel sum calculation that demonstrates performance benefits

Key technical features:

- Uses standard C++11 threading primitives: `std::thread`, `std::mutex`, `std::condition_variable`
- Implements task submission with `std::future` for retrieving results
- Handles proper synchronization between threads
- Uses `std::atomic` for thread-safe accumulation
- Automatically determines optimal thread count based on hardware

The code demonstrates core threading concepts like task queuing, worker management, thread synchronization, and graceful shutdown. You can easily extend this threadpool for your specific applications.