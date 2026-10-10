# Concise Control Flow with `if let`

You can think of `if let` as syntax sugar for a `match` that runs code when the value **matches only one pattern** and then **ignores all other values**.

## Classic Match Expression

```rust
let config_max = Some(3u8);
match config_max {
    Some(max) => println!(
        "The maximum is configured to be {}", max
    ),
    _ => (),
}
```

> [!NOTE]  
> The `Option<T>` `enum` has only two variants, but you can use the `if let` expression even if you have an `enum` with multiple variants as long as you only want to get **one specific pattern value**.

## Using `if let`

This code is equivalent to the example code above, it just has a shorter, more concise syntax.

```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
}
```

## Using `if let` with `else`

In the second example above we only want to run a block of code if our chosen single pattern matches the expression, but using `if let ... else` will enable us to run another block of code if it didn't match the single pattern that we wanted to match with.

```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
} else {
    ()
}
```