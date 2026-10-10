# Writing Meta Functions

Algorithm as a meta function that will operate on a `std::tuple` (from `#include <tuple>`), a heterogeneous container (can hold elements of different types). It is a variadic template that can hold any number of template parameters.

# The Standard Library Tuple

```c++
#include <iostream>
#include <utility>
#include <tuple>

int main() {
    std::tuple<int, bool, float> tup {1, false, 3.f};

    // using indices, returns a reference to the
    // element at that index.
    std::get<2>(tup) = 1.5f;

    // In C++14+ we can use the type to get the element
    // in a tuple, this function form will not compile
    // if the tuple contains that type more than once.
    std::get<float>(tup) = 1.5f;

    // std::tuple_element returns the type of an
    // element in a certain index.
    std::tuple_element<2, decltype(tup)>::type squared = std::get<2>(tup) * std::get<2>(tup);

    std::cout << "square float = " << squared << '\n';
}
```

# Dependent Name

A **name used** in a **template declaration** or **definition** and is **dependent on a template-parameter** is assumed _**not to be a type**_.

Unless the applicable name lookup finds a type name (_calling the template-id or type-name-id directly, and not through a template parameter_), or the name is qualified by the keyword `typename`.

- The **compiler needs to know** if it is **dealing** with a **type** or a **value**.

    ```c++
    template<typename T>
    struct Foo {
        // compilation error (before C++20), the compiler will assume
        // that `name_of_type` is a value.
        T::name_of_type b; 
    };

    template<typename T>
    struct Foo {
        // compilation success, the compiler knows that
        // `name_of_type` is indeed a type.
        typename T::name_of_type b;
    };
    ```

    (NOTE: C++20 relaxed this rule, `typename` became optional in some contexts where only a type can appear, like member declarations or return types, but it is still required in many places, so it's safer to write it.)

- The **compiler needs to know** if it is **dealing** with a **template** or a **value**.

    ```c++
    template<typename T>
    void foo() {
        // compilation error, the compiler does not know that
        // `T::function` is a template, so it parses `<` as "less than"
        T::function<int>(); 
    }

    template<typename T>
    void foo() {
        // compilation success, the compiler now knows that
        // `T::function` is a (static member function) template.
        T::template function<int>(); 
    }

    // we also need the `template` keyword when calling a member template
    // through an instance of `T` or a pointer to `T`
    template<typename T>
    void foo(T obj, T* ptr) {
        obj.template function<int>(); 
        ptr->template function<int>(); 
    }

    // and when a dependent member template is used as a TYPE we need both keywords
    template<typename T>
    void bar() {
        typename T::template type<int> x;
    }
    ```

# Exceptions to Specifying Dependent Name With Keywords

- When **specifying the bases** of which a class inherits.

    In this example, since **inheriting** is always only possible with types, there is actually no need to use the `typename` keyword here, in fact using it would cause a compilation error.

    ```c++
    // compilation error
    template<typename T>
    struct Derived : typename T::type {};

    // compilation success
    template<typename T>
    struct Derived : T::type {};
    ```

    Though there will be some cases where we would have to **nest templates** or **types** inside the template when inheriting to a derived class, in those cases, inside the `<>` we would need to use the `typename` or the `template` keyword.

-----

- Another example is, when the **template-id** is the one being referred to in the derived class's `using`-directive.

    Here a using-declaration names the member itself (not a specialization of it), so `Base::type` can be brought into `Derived` without the `template` keyword; writing `template` there is actually a syntax error.

    ```c++
    struct Base {
        template<typename T>
        struct type {};
    };

    // compilation error
    struct Derived : Base {
        using Base::template type;
    };

    // compilation success
    struct Derived : Base {
        using Base::type;
    };
    ```

# Meta-Programming an IF/ELSE meta-function

This is useful when dealing with **types** in general (the standard library version is `std::conditional<condition, Y, N>`), but doing this with values is mostly not recommended since we can just use a plain `if constexpr` for those cases.

```c++
/// \returns the `Y` type if condition is true.
/// \returns the `N` type if condition is false.
template<bool condition, typename Y, typename N>
struct if_;

template<typename Y, typename N>
struct if_<true, Y, N> {
    using type = Y;
};

template<typename Y, typename N>
struct if_<false, Y, N> {
    using type = N;
};
```

# Meta-Programming our `contains_type` meta-function

Here we will create a meta function called `contains_type` that will check all elements of a tuple if it contains a certain type, we will do this using recursion.

```c++
#include <tuple>
#include <type_traits>

template<typename find_t, typename TUPLE, size_t i = 0>
struct contains_type :
    if_<
        // is element's type on index 'i' equal to 'find_t' type?
        std::is_same<find_t, std::tuple_element_t<i, TUPLE>>::value,
        
        // yes: set `if_<>::type` equal to 'std::true_type'
        std::true_type,

        // no: nest another 'if_'
        typename if_<
            // is index 'i' equal to the last index value of the tuple?
            i == std::tuple_size<TUPLE>::value - 1,

            // yes: set `if_<>::type` equal to 'std::false_type'
            std::false_type,

            // no: check the next index by recursion
            contains_type<find_t, TUPLE, i + 1>
        >::type
    >::type
{};

/// specialize for empty tuple
template<typename find_t>
struct contains_type<find_t, std::tuple<>> : std::false_type {};
```

# Using our Meta-Function

```c++
int main() {
    std::tuple<int, bool, float> tup;

    std::cout << std::boolalpha << contains_type<bool, decltype(tup)>::value << '\n'; // true

    std::cout << std::boolalpha << contains_type<double, decltype(tup)>::value << '\n'; // false

    std::cout << std::boolalpha << contains_type<bool, std::tuple<>>::value << '\n'; // false
}
```