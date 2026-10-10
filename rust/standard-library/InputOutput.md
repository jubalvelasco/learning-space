# Rust I/O

## Standard Input

`read_line` takes whatever the user types into standard input (including the trailing newline) and appends it to a string (without overwriting its contents). It returns a `Result` (the number of bytes read), so you usually call `.expect(...)` on it.

The string argument needs to be mutable so the method can change the string’s content.

```rust
std::io::stdin().read_line(&mut variable).expect("Failed to read line");
```

## Output

Only **variable names** (identifiers) are **allowed** to be placed inside the `{}` when using print macros (inline format args, Rust 2021+).

On the other hand, **Expressions** are not allowed to be placed inside the **placeholder** (`{}`) instead we leave the curly-braces empty, then we pass the expression in the succeeding parameters.

The number and order of the empty curly-braces inside the `""` and the succeeding expressions passed to the succeeding parameters should match each other.

```rust
// macro for printing with new-line
println!("this variable is equal to = {variable}");

// without new line
print!("this expression {x} + {y} = {}", x + y);
```

