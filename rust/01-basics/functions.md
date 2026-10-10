# **[Rust Lang](../Rustlang.md)**

# **Functions**

In rust functions are created using the **`fn`** keyword, the very first function that users see in rust is the main driver function.

### **The main driver function**

This is the entry point of a rust program, of an **executable rust crate**.

```rust
// src/main.rs
fn main() {
    println!("Hello, world!");
}
```

Rust does not need forward declarations (unlike C/C++). It's not really "hoisting" like in JavaScript, items (functions, structs, etc.) in a module are simply all visible to each other regardless of the order they are written in,
so the example code below works even though `greet_world` is defined after the main function.

Rust doesn’t care where you define your functions, only that they’re defined somewhere in a scope that can be seen by the caller.

```rust
fn main() {
    greet_world();
}

fn greet_world() {
    println!("Hello world, I'm from a rust function");
}
```

### **Function Parameters**

Like in C and C++ in rust when we create a function, the **parameter**'s signature should always have a **type annotation**, it will not compile if it doesn't.

> [!IMPORTANT]  
> Values whose types do not implement the `Copy` trait (e.g. `String`, `Vec`, which own heap memory) that are passed into the function parameter as an argument are moved by default.
>
> This behavior will make the original variable unusable, hence in order to use it back outside of the function scope we either need to [return it back](../02-ownership-borrowing-references/01-ownership.md#ownership-rules), or use [references to borrow it](../02-ownership-borrowing-references/02-borrowing-and-reference.md).

```rust
fn foo(bar: i32) {
    println!("function parameter = {bar}");
}
```

- **parameters** - part of the function signature.
- **argument** - the value you pass into the parameters of the function.

But these two terms are often mixed up, and people use them interchangeably in casual conversations.

# **Statements**

In rust, **Statements** are instructions that perform some action and **do not return** a **value**.

Statements do not return values. Therefore, you can’t assign a let statement to another variable :

```rust
fn main() {
    // error!
    let x = (let y = 6);
}
```

So something like `int x = (y = 6);` in C and C++ has no equivalent with `let` in rust, the inner `let y = 6` is a statement hence it has no return value.

# Expressions

An **Expression** in rust always evaluates to a value, we can use expressions inside statements and we can compose statements using expressions:
- **calling functions** are expressions in rust.
- **calling macro** are expressions in rust.
- **new scope block** created with curly brackets is an expression :
    ```rust
    fn main() {
        let y = {
            let x = 10;
            x + 1
        };
        println!("The value of y is: {y}");
    }
    ```

    **Notice** here that the `x + 1` does not have a **semicolon**, if you add a semicolon in the last line of a new block, that block will become a **statement** not an expression, keep that in mind.

### **Functions with return values**

Functions that return a value in rust should always have a return type annotation, we do this using the `->` arrow operator. We could also **omit** a **semicolon** after a **return statement**, but we can add one if we want to.

```rust
fn cubic_function(x: i64) -> i64 {
    return x * x * x
}
```

Though in rust, the **final expression** of a function body (the last line without a semicolon) is automatically the **return value**, hence we can **omit** the **return** keyword in the last line of a function, as long as we don't put a semicolon `;` after it.

The `return` keyword is useful when we need to return early from a function.

```rust
fn relu_activation(x: i64) -> i64 {
    if x > 0 {
        return x // returns the value of x
    }
    777
}
// for x <= 0 this function implicitly returns the value 777.
```

If semicolons are added in the last line it will become a statement, hence there will be no return value at all, actually it will not even compile.

```rust
fn relu_activation(x: i64) -> i64 {
    if x > 0 {
        return x // returns the value of x
    }
    777;
}
// will throw a compilation error.
```

### Associated Functions

A function that’s implemented on a type (inside an `impl` block, without a `self` parameter), called using the `::` path separator, e.g. `String::new()`.