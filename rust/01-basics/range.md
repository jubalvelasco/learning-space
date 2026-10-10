# Range

**Creating a range**

```rust
let teen_age = 13..=19;
```

**Using range in for loops**

```rust
for age in teen_age {
    print!("{age} ");
}
println!("");

// or

for num in 13..=19 {
    print!("{num} ");
}
println!("");
```

**Output**

```shell
13 14 15 16 17 18 19 
13 14 15 16 17 18 19 
```

NOTE: `13..=19` is an **inclusive** range (includes 19), while `13..19` is **exclusive** (stops at 18).