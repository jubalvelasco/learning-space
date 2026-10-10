# **Concurrency**

- **Concurrency** - two or more tasks making progress during overlapping time periods (they may be interleaved on a single core, switching back and forth)
- **Parallelism** - two or more tasks literally executing at the same time (requires multiple cores/processors).


## **C++11 - memory model**

- Now a program consists of one or more _**threads of execution**_.
- Every write to a single memory location must be ordered (via a _**happens-before**_ relationship, built from _**synchronizes-with**_ relationships) with respect to all other reads and/or writes of that memory location from other threads, or else the program has a data race, which is undefined behavior (unless all those accesses are atomic).
- _**Synchronizes-with**_ relationships can be established by using various standard library facilities, such as `std::mutex` and `std::atomic<T>`.


## **C++11 - starting a new thread**

```c++
#include <thread>
```

In C++11, the standard library "owns" the notion of creating new threads.
To create a thread, you create a `std::thread` object. The constructor argument is a callable that says what you want the thread to do.

```c++
std::thread threadB = std::thread([](){
    puts("Hello from thread B!");
});
```

## **Joining the finished threads**

The new thread starts executing immediately. A `std::thread` object that
was constructed with a function is _**joinable**_ from the start (whether
its job is already finished or not), until `.join()` or `.detach()` is called.

Call `.join()` (or `.detach()`) on a `std::thread` object before destroying it,
destroying a still-joinable `std::thread` calls `std::terminate()`.
The `.join()` call will _**block**_, if necessary, until the other thread's job
is finished.

```c++
puts("Hello from thread A!");
threadB.join();
```

## **Getting the "result" of a thread**

We don't need any special way to return an "exit status" from a thread,
because joining with a child thread is a synchronizing operation.

```c++
int result = 0;
std::thread threadB = std::thread([&](){
    puts("Hello from thread B!");
    result = 24;
});

puts("Hello from thread A!");
threadB.join();
printf("The result of threadB was %d\n", result);
```

The `result` read in `printf` synchronizes with
write on the `threadB`: `result = 24`,
this is because the synchronizing operation `join()` was
executed/called/invoked after write and before the read,
they were separated by the synchronizing operation `join()`.

## **An example of data race on an int**

More specifically an example of a **physical data race**.

```c++
using SC = std::chrono::steady_clock;
auto deadline = SC::now() + std::chrono::seconds(10);

int counter = 0;

std::thread threadB = std::thread([&](){
    while(SC::now() < deadline) {
        printf("B: %d\n", ++counter);
    };
});

while(SC::now() < deadline) {
    printf("A: %d\n", ++counter);
}

threadB.join();
```

This is a **physical data race** because both the threads
might read/write the `counter` simultaneously, this is
undefined behavior.

## **Fixing the physical data race via atomic**

By changing the declaration of the `counter` with atomic,
we can fix the physical data race, and remove the
undefined behavior.

```c++
std::atomic<int> counter = 0;
```

This is because `std::atomic` synchronizes with every other access to it.

**NOTE:** Changing the `int counter = 0` to
`std::atomic<int> counter = 0` will only fix the **physical data race**,
in some programs (not all) the previous code might still be considered
as a bug since it still contains other types of data race like:
**semantic data race** and **logical data race** (race conditions in the program's logic), but these data races
are not considered **undefined behavior**.

## **Fixing the physical data race via mutex**

When a mutex is locked, any thread attempting to lock the mutex will pause its execution until the mutex is unlocked. Once the mutex is unlocked, a waiting thread will obtain the lock and continue.

```c++
using SC = std::chrono::steady_clock;
auto deadline = SC::now() + std::chrono::seconds(10);

int counter = 0;
std::mutex mtx;

std::thread threadB = std::thread([&](){
    while(SC::now() < deadline) {
        mtx.lock();
        printf("B: %d\n", ++counter);
        mtx.unlock();
    };
});

while(SC::now() < deadline) {
    mtx.lock();
    printf("A: %d\n", ++counter);
    mtx.unlock();
}

threadB.join();
```

## **Lock Guards**

_**Mutex'es**_ can produce **"deadlocks"** if we forget to
call `.unlock()` (e.g. an early `return`, or an exception thrown between `lock()` and `unlock()`),
since every other thread that tries to lock it will wait forever (also, destroying a mutex that is
still locked is undefined behavior). A way to fix
this problem is to use a **lock guard** that will unlock
a mutex automatically when that lock guard goes out of
scope (RAII), declare the lock guard in the scope that needs the lock. eg:

```c++
std::lock_guard<std::mutex> lock_g(mtx);
```

In C++17 `std::scoped_lock` is introduced, it is the same
as `std::lock_guard` except that it can
take multiple mutex'es, and locks them all using a deadlock-avoidance algorithm (like `std::lock`).

Another classic cause of deadlock is two threads locking two mutexes in opposite orders
(thread 1 locks A then B, thread 2 locks B then A); always lock in the same order or use `std::scoped_lock`.

_**Mutex'es**_ can also produce **"starvation"** where
the other threads cannot acquire resource or is
waiting for a very, very long time. A possible cause
of this case is when one thread that locks a mutex takes
an eternity before unlocking it back.

The `std::unique_lock` is a bit more powerful than
`std::lock_guard` and `std::scoped_lock` in that we can manually control
locking and unlocking (and it is movable), and it is also required by `std::condition_variable`. It also follows the same RAII.
