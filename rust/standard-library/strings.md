# Strings

### UTF-8 Strings

**Create a string using the `String` type**

```rust
// UTF-8 string
let mut s = String::new(); // must be `mut` to be able to append to it
let name = String::from("from a hard-coded string");
```

**Append to string**

```rust
s.push_str("string literal");
s.push_str(&heap_string);   // &String coerces to &str
s.push_str(str_slice);      // a &str
s.push('c');                // a single char
```

**Remove front and back white spaces**

```rust
let trimmed: &str = s.trim(); // returns a new string slice, `s` itself is not modified
```

**Converting to other types**

The `.parse()` method uses the type annotation `usize` (type inference) to know which type to convert to. It returns a `Result`, since the conversion can fail.

```rust
let str_idx = String::from("102");
let index: usize = str_idx
    .trim()
    .parse()
    .expect("Index entered was not a number");
```