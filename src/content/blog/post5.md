---
title: "Rust - Basic"
description: "Rust Programming notes for SRM Vadapalani"
pubDate: "Aug 21, 2024"
heroImage: "/rust.jpg"
tags: ["RUST", "SRM"]
---
# Rust Programming Notes

## 1. Rust - Introduction

- Systems programming language focused on safety, concurrency, and performance
- Developed by Mozilla Research
- Key features: memory safety without garbage collection, concurrency without data races
- Compiler: rustc
- Package manager: Cargo

Example: Hello World program
```rust
fn main() {
    println!("Hello, world!");
}
```

## 2. Variables & Mutability

- Variables are immutable by default
- Use `mut` keyword to make variables mutable
- Constants are always immutable and declared with `const`

Examples:
```rust
let x = 5; // Immutable
let mut y = 5; // Mutable
const MAX_POINTS: u32 = 100_000; // Constant
```

## 3. Data Types - Scalar and Compound

### Scalar Types

1. Integer: i8, i16, i32, i64, i128, isize, u8, u16, u32, u64, u128, usize
2. Floating-point: f32, f64
3. Boolean: bool
4. Character: char

Examples:
```rust
let a: i32 = -42;
let b: f64 = 3.14;
let c: bool = true;
let d: char = 'Z';
```

### Compound Types

1. Tuple: Fixed-length collection of values of different types
2. Array: Fixed-length collection of values of the same type

Examples:
```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
let arr: [i32; 5] = [1, 2, 3, 4, 5];
```

Operators:
- Arithmetic: +, -, *, /, %
- Comparison: ==, !=, <, >, <=, >=
- Logical: &&, ||, !
- Bitwise: &, |, ^, <<, >>

## 4. Functions and Comments

- Functions are declared using the `fn` keyword
- Parameters are specified with their types
- Return type is specified after `->`
- Comments: `//` for single-line, `/* */` for multi-line

Example:
```rust
// This function adds two numbers
fn add(a: i32, b: i32) -> i32 {
    a + b // Implicit return (no semicolon)
}

fn main() {
    let result = add(5, 3);
    println!("Result: {}", result);
}
```

## 5. Ownership and Borrowing Rules

Ownership:
- Each value has an owner (a variable)
- Only one owner at a time
- When owner goes out of scope, value is dropped

Borrowing:
- References allow you to refer to a value without taking ownership
- `&` for immutable references, `&mut` for mutable references
- You can have either one mutable reference or any number of immutable references

Example:
```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // Borrowing s1
    println!("The length of '{}' is {}.", s1, len);
    let s2 = s1;
    println!("lets print s2 - {}", s2);
    //will get errored
    println!("lets print s1 - {}", s1);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

## 6. Control Structures

1. if / if else:
```rust
let number = 6;
if number % 2 == 0 {
    println!("Even");
} else {
    println!("Odd");
}
```

2. loop:
```rust
let mut counter = 0;
loop {
    counter += 1;
    if counter == 10 {
        break;
    }
}
```

3. while:
```rust
let mut number = 3;
while number != 0 {
    println!("{}!", number);
    number -= 1;
}
```

4. for:
```rust
for i in 1..=5 {
    println!("Number: {}", i);
}
```

## Collections

### 7. Strings

- Two types: `String` (growable) and `&str` (string slice)
- `String` is heap-allocated and mutable
- `&str` is immutable and can be stack-allocated

Example:
```rust
let s1 = String::from("Hello");
let s2 = "World"; // &str
let s3 = s1 + " " + s2;
println!("{}", s3);
```

### 8. Vectors

- Growable array-like structure
- Stores values of the same type

Example:
```rust
let mut v: Vec<i32> = Vec::new();
v.push(1);
v.push(2);
v.push(3);

for i in &v {
    println!("{}", i);
}
```

### 9. Hash Map

- Key-value pairs
- Keys must be unique

Example:
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Red"), 50);

match scores.get("Blue") {
    Some(score) => println!("Blue team score: {}", score),
    None => println!("Blue team not found"),
}
```