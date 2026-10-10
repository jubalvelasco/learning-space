# Modules

By default, Rust has a set of items defined in the standard library that it brings into the scope of every program. This set is called the prelude, and you can see everything in it in the standard library documentation.

### Using a Module

This also recursively applies to submodules which are like modules used inside another module.

```rust
mod garden;

// or, you could also make the module public so that it can
// be used by external code

pub mod garden;
```

This in a sense is equivalent (but not really) to:

```c++
// in C++
#include <garden>
```

Or equivalent to the `import` keyword in other languages.

### The Root Library Crate Module

The `src/lib.rs` file is the root of the library crate. Its items are not accessed with a `mod lib;` declaration, so calling its function `lib::your_function();` will not work since it is already the root module of a separate crate.

Instead, from the binary crate (`src/main.rs`) you need to use the name of your crate (package name), and from inside the library itself you use `crate::`, e.g.;

```rust
crate_name::your_function(); // from src/main.rs
crate::your_function();      // from inside the library crate
```

### The `use` keyword: Using types directly.

```rust
use std::io;
```

This is roughly equivalent to the following in C++ (pseudo-code, `std::io` is not a real C++ namespace).

```c++
namespace io = std::io; // brings `io` into scope (namespace alias)
// or
using namespace std;    // uses everything including the `io` (closer to a glob import `use std::*;`)

// usage
io::some_function();
```

### The `use` keyword: Giving names or aliases to types.

```rust
use std::io as IO;
```

This is equivalent to C++'s namespace aliasing (or type aliasing when the item is a type).

```c++
namespace IO = std::io;  // for modules/namespaces
// or, for types
using IO = std::some_type;

// usage
IO::some_function();
```

### The `use` keyword: Scope

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

The examples above use the `use` keyword to create shortcuts to the items in a module, but those shortcuts will **only apply in the current scope** of a module, the `use` keyword will not propagate into the parent module scope obviously like any other languages, but the most important thing to remember is that, it **also does not apply** to the **inner modules scope** (use `super::hosting` there, or move the `use` inside the inner module).

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

mod customer {
    pub fn eat_at_restaurant() {
        // The compiler error shows that the shortcut
        // no longer applies within the customer module
        hosting::add_to_waitlist();
    }
}
```
# The `use` keyword : Idiomatic Widely Accepted Convention

There’s no strong reason behind these idioms: it’s just the convention that has emerged, and folks have gotten used to reading and writing Rust code this way.

- For **Functions**, the convention is to not create a shortcut directly to the function with the `use` keyword, instead bring only its **parent module** into scope (e.g. `use crate::front_of_house::hosting;` then call `hosting::add_to_waitlist()`), so it's clear the function isn't locally defined.

- For **Structs**, **Enums** and **other items** on the other hand, people normally create direct shortcuts when using them from a module (e.g. `use std::collections::HashMap;`).

# Re-Exporting Names with `use` Keyword

**lib.rs**
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;
```

**main.rs**
```rust
restaurant::hosting::add_to_waitlist(); // `restaurant` is the crate (package) name
```

# Using Nested Paths to Clean Up Large use Lists

We can combine multiple shortcuts created by `use` keyword into one line using **nested paths**.

**Example 1 : Multiple Shortcuts In Different Lines**
```rust
use std::cmp::Ordering;
use std::io;
```

**Example 1 : Multiple Shortcuts In One Line**
```rust
use std::{cmp::Ordering, io};
```

**Example 2 : Multiple Shortcuts In Different Lines**
```rust
use std::io;
use std::io::Write;
```

**Example 2 : Multiple Shortcuts In One Line**

We could also use the `self` keyword here to bring in `std::io` itself (the module named by the path), which is how this differs from example 1.

```rust
use std::io::{self, Write};
```

# The Glob Operator `*`

Like a **wildcard** the glob `*` operator brings all of the public items in a specific module scope into the current scope.

```rust
use std::collections::*;
```