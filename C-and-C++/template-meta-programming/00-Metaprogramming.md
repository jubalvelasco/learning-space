# Metaprogramming

- **Program** - Something that takes input data, and transforms/processes that data into output data or information.

- **Metaprogram** - Something that treats programs or code (or types) as its data: it generates, transforms, or inspects code instead of (or in addition to) ordinary data.
    - If a program looks at (and possibly modifies) its own structure it is called **reflection**. C++ template metaprogramming is **not** really reflection: it is compile-time computation over types and constants (the compiler "runs" the template code while compiling). Real static reflection is only being added in C++26.

# Template Specialization

- **Basic Template Declaration**

    ```c++
    template<typename KEY, typename DATA>
    struct Qmap {
        Qmap() { std::cout << "Qmap\n"; }
    };
    ```

- **Partial Specialization** - not all template parameters are filled in but only some, it will work for all cases of the template where the first parameter is an `int`. (Only class templates and variable templates can be partially specialized, function templates cannot.)

    ```c++
    template<typename DATA>
    struct Qmap<int, DATA> {
        Qmap() { std::cout << "Qmap<int, DATA>\n"; }
    };
    ```

- **Full Specialization** - all template parameters are filled in.

    ```c++
    template<>
    struct Qmap<int, int> {
        Qmap() { std::cout << "Qmap<int, int>\n"; }
    };
    ```

The compiler will check all of the different specializations, as well as the main template definition, and see which ones will work with these template parameters, the compiler will then choose which of the **candidate set** is more specific.

# Initializing with Template Classes - Template Argument Deduction

- **Aggregate Initialization** - providing values inside `{}` that matches the member variables values and types of the class.

- **Constructor Initialization** - construction of objects with `{}` is also possible by creating a constructor that will match the values and types inside the `{}` initialization values.

```c++
int main() {
    // chooses the main template definition
    auto A = Qmap<float, bool>();

    // the main and partial definition are a match, but the compiler
    // will choose the partial template specialization because it is
    // more specific.
    auto B = Qmap<int, bool>();

    // will choose the full specialization `Qmap<int, int>`
    // since it is the most specific.
    auto C = Qmap<int, int>();
}
```

# Type Traits

C++ provides predefined meta functions built in to the standard library.

# Checking if Type is a Pointer

To know more about template meta programming let us implement the `std::is_pointer<T>::value` of the standard library that checks if the given template parameter `T` type is a pointer type `T*` or not.

```c++
// for all possible cases except the Specializations, this struct will
// return false
template<typename T>
struct is_pointer {
    static constexpr bool value = false;
};

// specializes if the given type is a pointer it will return true.
template<typename T>
struct is_pointer<T*> {
    static constexpr bool value = true;
};

// NOTE: this simple version returns false for cv-qualified pointers like `int* const`,
// the real `std::is_pointer` also handles those (it removes cv-qualifiers first).
```

# Manipulating Types: Stripping Pointer Types

Here is another example of how to get only the type `T` of a pointer `T*` using template specialization.

```c++
// returns the type itself if the type is not a pointer.
template<typename T>
struct strip_pointer {
    using type = T;
};

// returns only the type of a given pointer type.
template <typename T>
struct strip_pointer<T*> {
    using type = T;
};

// the standard library version of this is `std::remove_pointer<T>::type`
```