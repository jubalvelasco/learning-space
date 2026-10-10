# **Concurrency 2**

## **Threads**

One process can have many threads.

- static local variables are initialized only once, even if multiple threads reach their declaration at the same time (this initialization is thread-safe since C++11). Use `thread_local` if you want one variable per thread.
- Each thread shares the same code, data, and kernel
context.
    - run-time heap
    - read/write data
    - read-only code/data
    - shared libraries
- A thread has its own thread ID (TID).
- A thread has its own logical control flow.
- A thread has its own stack for local variables.

## **Passing values on a thread**

- If the `function` or lambda has a **pass-by-value** parameter.

    ```c++
    std::thread new_t(function, n);
    ```

- If the `function` or lambda has a **pass-by-reference** parameter.

    ```c++
    std::thread new_t(function, std::ref(n));
    ```

- Passing a `method` of a specific object (instance of a class).

    ```c++
    std::thread new_t(&class_name::method, &instance);
    ```

- Moving a thread.

    ```c++
    std::thread new_t(std::move(old_thread));
    ```

## **Executing multiple threads**

```c++
std::vector<std::thread> threads;

auto lambda = [](size_t x) {
    std::cout << "lambda.thread.get_id(" << x << ") : " << std::this_thread::get_id() << "\n";
};

for (size_t i = 0; i < 10; ++i) {
    threads.push_back(std::thread(lambda, i));
}

for (size_t i = 0; i < threads.size(); ++i) {
    threads[i].join();
}
```
## **C++20 jthreads**

In C++20 `std::jthread` is introduced where it automatically
calls `.request_stop()` and then `.join()` when the destructor is called, this happens
when the `jthread` object goes out of scope.

```c++
std::vector<std::jthread> threads;

auto lambda = [](size_t x) {
    std::cout << "lambda.thread.get_id(" << x << ") : " << std::this_thread::get_id() << "\n";
};

for (size_t i = 0; i < 10; ++i) {
    threads.push_back(std::jthread(lambda, i));
}

// in this case we don't need to create a loop that
// will call the .join() method for each thread
```

## **Condition Variables**

A way to signal an event between 2 or more threads.

Condition variables allow us to, in a sense, put a thread to sleep
(waiting, without busy-looping), until it is notified that it should start 
working.

```c++
// 42:49 - Back to Basics - Mike Shah
std::mutex mtx;
std::condition_variable cv;
bool ready = false;

std::thread worker([&]() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [&]{ return ready; }); // sleeps until notified AND ready == true (handles spurious wakeups)
    puts("worker: got the signal");
});

{
    std::lock_guard<std::mutex> lock(mtx);
    ready = true;
}
cv.notify_one(); // wake up one waiting thread (notify_all() wakes all)
worker.join();
```

## **Semaphores, Latches and Barriers (C++20)**

- `std::counting_semaphore` - allows at most N threads to access a resource/critical section at the same time.
- `std::latch` - a single-use countdown: threads wait until the counter reaches zero.
- `std::barrier` - a reusable synchronization point: threads wait until a fixed number of threads have arrived, then all continue (and the barrier resets for the next phase).

## **Multiple threads using Asynchronous Programming**

Asynchronous programming does not mean multi-threading in general
but in C++ we can also create multiple threads with it.

We can start running a function simultaneously using `std::async`.

```c++
auto foo = std::async(&function, arg1, arg2, ..., argN);
```

And like `std::thread`'s `.join()`, we call `.get()` on the returned `std::future` to wait for the result (synchronize).

```c++
auto foo1 = std::async(&function, arg1, arg2, ..., argN);
auto foo2 = std::async(&function, arg1, arg2, ..., argN);

auto foo1_result = foo1.get();
auto foo2_result = foo2.get();
```
Passing `std::launch::async` as the first argument when constructing
`std::async` will force the program to create a new thread for that
function, if you omit this the decision of whether to create a thread
or not will depend on the underlying implementation (with `std::launch::deferred` the function
runs lazily in the calling thread when `.get()`/`.wait()` is called).

NOTE: the `std::future` returned by `std::async` blocks in its destructor until the task finishes,
so if you don't store it (`std::async(...);`), the call effectively runs synchronously.

```c++
auto foo = std::async(std::launch::async, &function, arg1, arg2, ..., argN);
```

One advantage of `std::async` is that by calling `.get()` 
method we can get the return value of the function, this
is not the case for `std::thread`'s `.join()` method.