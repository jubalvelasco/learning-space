# Smart Pointers

```c++
#include <memory> // <- access all smart pointers
```

## unique_ptr

```c++
std::unique_ptr<myClass> myObj = std::make_unique<myClass>();
```

- this smart pointer automatically calls `delete` (or `delete[]` for arrays) on the heap allocated object/array when the pointer goes out of scope
- unique pointers cannot be copied to another unique_ptr or other smart pointers (they can only be moved with `std::move`), but you can get a non-owning raw pointer to the object using the ```.get()``` method

```c++
#include <iostream>
#include <memory>

int main()
{
    std::unique_ptr<int[]> arr1(new int[10]); // <- declaration, you might prefer `std::make_unique<int[]>(10)` (C++14) to be safer
    std::unique_ptr<int[]> arr2 = arr1; // <- compile error (copy is deleted), remove this line to compile the program
    int* arr3 = arr1.get(); // <- allowed (arr3 does not own the memory, never delete it)

    for(size_t i=0; i<10; ++i)
        arr1[i] = i+1;

    arr3[2] = -2;

    for(size_t i=0; i<10; ++i)
        std::cout << arr1[i] << " ";

    std::cout << "\n";
    return 0;
}

```

if you compile this with ```-fsanitize=address```, even though we did not ```delete``` the allocated array, no memory leak will be reported

output: ```1 2 -2 4 5 6 7 8 9 10```

## shared_ptr

- this smart pointer keeps a reference count and only calls delete (frees the object) when the reference count goes to zero

```c++
#include <iostream>
#include <memory>
#include <string>

class Subject
{
    public:
        std::string name;
        short credits;
        int* arr;
        Subject(std::string name, short credits) : name(name), credits(credits), arr(new int[10])
        {
            for(size_t i=0; i<10; ++i) arr[i] = i;
        }
        ~Subject()
        {
            std::cout << "\tDestroyed = \n";
            display();
            delete [] arr; // free the array in the destructor only, NOT in display(), otherwise calling display() twice would double free
        }
        void display()
        {
            std::cout << "\tsubject name : " << name << "\n"
                      << "\tcredits      : " << credits << "\n\t";
            for(size_t i=0; i<10; ++i) std::cout << arr[i] << " ";
            std::cout << "\n";
        }
};

int main()
{
    std::cout << "Enter main scope \n";
    {
        std::cout << "\tEnter inner scope 1 \n";
        std::shared_ptr<Subject> outside_ptr;
        {
            std::cout << "\t\tEnter inner scope 2 \n";
            std::shared_ptr<Subject> math = std::make_shared<Subject>("Math Modern World",3);
            
            outside_ptr = math;

            std::cout << "\t\tExit inner scope 2 \n";
        }
        std::cout << "\tExit inner scope 1 \n";
    }
    std::cout << "Exit main scope \n";
    return 0;
}

```

output
```
Enter main scope 
        Enter inner scope 1 
                Enter inner scope 2 
                Exit inner scope 2 
        Exit inner scope 1 
        Destroyed = 
        subject name : Math Modern World
        credits      : 3
        0 1 2 3 4 5 6 7 8 9 
Exit main scope 
```

# Smart pointers in C++17 - what they are, pros/cons, and when to use them

---

## The big three in C++17

### `std::unique_ptr<T>`

**What it is:** exclusive-ownership smart pointer. Non-copyable, movable. Single owner; when it is destroyed, the object is destroyed.

**Syntax / common patterns**

```c++
auto p = std::make_unique<Foo>(args);        // preferred
std::unique_ptr<Foo> p2 = std::move(p);      // transfer ownership
std::unique_ptr<Foo[]> arr = std::make_unique<Foo[]>(n); // array specialization
```

**Pros**

* Zero/low overhead (no reference counting).
* Clear ownership semantics: exactly one owner.
* Prevents leaks when used consistently.
* `make_unique` is exception-safe and efficient.

**Cons**

* Cannot be shared (no copy), so not usable where multiple owners are required.
* You must move to transfer ownership, which is fine but sometimes inconvenient when many parties need access.

**When to use**

* Default choice for dynamically-allocated objects when only one owner is required.
* Use for RAII wrappers, pimpl, resources with clear single ownership.

---

### `std::shared_ptr<T>`

**What it is:** shared ownership using reference counting. Many `shared_ptr`s can own the same object; the object is destroyed when the last `shared_ptr` is destroyed.

**Syntax / common patterns**

```c++
auto p = std::make_shared<Foo>(args);   // preferred: one allocation for control block + object
std::shared_ptr<Foo> p2 = p;            // copy increments use_count
p.reset();                              // decrement count; destroys object if count==0
```

**Pros**

* Easy to share ownership across functions, objects, threads.
* Well supported: custom deleter, aliasing constructor, `enable_shared_from_this`.
* `std::make_shared` is efficient (single allocation for control block + object).

**Cons**

* Runtime overhead: atomic (or at least thread-safe) increments/decrements of reference count.
* Slight memory overhead for control block.
* **Risk of reference cycles** (A → B → A) causing memory leaks — must break cycles (usually via `weak_ptr`).
* Semantics can be abused (using shared\_ptr everywhere hides true ownership design).

**Thread-safety note:** increment/decrement of the control block is thread-safe (you can copy and destroy shared\_ptrs concurrently), but simultaneous access/modification of the pointed-to object still requires your synchronization.

**When to use**

* When multiple independent owners must share lifetime of an object.
* When objects are managed by different subsystems and lifetime must be automatic.

---

### `std::weak_ptr<T>`

**What it is:** a non-owning observer for an object managed by `shared_ptr`. It does **not** increase the reference count. You `lock()` a `weak_ptr` to obtain a `shared_ptr` safely (which returns an empty `shared_ptr` if the object is gone).

**Syntax / common patterns**

```c++
std::weak_ptr<Foo> w = p;            // p is shared_ptr<Foo>
if (auto sp = w.lock()) {           // sp is shared_ptr<Foo>
  sp->doSomething();
} else {
  // object already destroyed
}
```

**Pros**

* Breaks reference cycles when placed on one side of the cycle.
* Lets you test for lifetime without extending it.

**Cons**

* Slight overhead to lock and check expiration.
* Only meaningful if there is `shared_ptr` ownership somewhere.

**When to use**

* To observe a `shared_ptr` managed object without keeping it alive.
* To break cycles or provide caches/registries that don’t own objects.

---

## Things that are *not* in C++17 (or are deprecated/removed) — what to avoid

* `std::auto_ptr` — **deprecated in C++11 and removed in C++17**. Don’t use it. It had confusing ownership semantics (copy transferred ownership) and was replaced by `unique_ptr`.
* `std::observer_ptr` — only exists as `std::experimental::observer_ptr` (Library Fundamentals TS v2), **not part of standard C++17**. Use raw pointer or `weak_ptr`/`std::reference_wrapper` depending on intent.

---

## Useful supporting APIs / idioms

* `std::make_unique<T>(...)` and `std::make_shared<T>(...)` — prefer these for safety and efficiency.
* `std::enable_shared_from_this<T>` — lets an object safely obtain a `shared_ptr` to itself:

  ```c++
  struct X : std::enable_shared_from_this<X> {
    std::shared_ptr<X> getptr() { return shared_from_this(); }
  };
  ```
* `shared_ptr` aliasing constructor: lets a `shared_ptr` share ownership of an object but point to a subobject (useful for containers/parts).
* Custom deleters: supported by both `unique_ptr` (type part of pointer) and `shared_ptr` (stored in control block).

* `weak_ptr::lock()` vs. `weak_ptr::expired()`:

  * `.lock()` → attempts to promote the `weak_ptr` to a `shared_ptr`. If the object still exists, you get a valid `shared_ptr` you can use; if it’s already destroyed, you get an empty `shared_ptr`.
  * `.expired()` → just a check that returns `true` if the object no longer exists. It does **not** give you access to the object.
  * **When to use which:** use `.lock()` whenever you plan to *access* the object, since it safely gives you a temporary owning pointer. Use `.expired()` if you only need to test whether the object is still alive, but don’t need to actually use it.

---

## Performance & thread-safety summary

* `unique_ptr` — no reference-count overhead; fastest.
* `shared_ptr` — extra memory and runtime cost for reference count; copying is heavier.
* Control-block operations for `shared_ptr` are safe to call concurrently (atomic updates), but modifying the pointee still needs explicit synchronization.
* If you need atomic load/store of `shared_ptr` across threads, use the **atomic helper functions** in `<memory>`: `std::atomic_load`, `std::atomic_store`, `std::atomic_compare_exchange_weak/strong`, etc. (`std::atomic<std::shared_ptr<T>>` only exists since C++20, which also deprecates these helper functions.)

---

## Ownership guidelines — practical rules you can follow

1. **Prefer automatic storage (stack values) for simple objects.** No pointers needed if you can keep objects by value.
2. **Default to `std::unique_ptr` for heap objects.** It expresses ownership clearly and is cheap.
3. **Use `std::shared_ptr` only when ownership must be shared.** Think carefully — shared ownership hides lifetime and costs money.
4. **Use `std::weak_ptr` to observe or to break cycles created by `shared_ptr`.**
5. **Never use `std::auto_ptr` — it’s removed.**
6. **Prefer `make_unique` and `make_shared`** over raw `new`.
7. **Avoid raw owning pointers.** A raw pointer can be a non-owning observer; if you mean ownership, use a smart pointer.
8. **If performance is critical and you need lower overhead, consider value semantics, `std::variant`, or intrusive reference counting (e.g., `boost::intrusive_ptr`) but only with care.**

---

## Quick decision cheat-sheet

* Want single ownership, transferable? → `std::unique_ptr`.
* Need shared ownership across multiple owners? → `std::shared_ptr` (+ `std::weak_ptr` to observe).
* Need to avoid keeping object alive but need to check existence? → `std::weak_ptr`.
* Need very low overhead or compile-time polymorphism? → prefer value objects, templates, `std::variant`, or virtual tables on stack-allocated objects (not smart pointers).
