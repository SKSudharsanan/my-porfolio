---
title: "Rust - Advanced"
description: "Rust Programming notes for SRM Vadapalani"
pubDate: "Aug 21, 2024"
heroImage: "/rust.jpg"
tags: ["RUST", "SRM"]
---
# Rust Programming Notes (Advanced Topics)

## 10. Struct

Structs are custom data types that let you group related data.

Example:
```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    println!("User {} has email {}", user1.username, user1.email);
}
```

## 11. Struct Implementation

You can implement methods and associated functions for structs.

Example:
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Method
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // Associated function
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    println!("Area: {}", rect1.area());

    let sq = Rectangle::square(3);
    println!("Square area: {}", sq.area());
}
```

## 12. Enum

Enums allow you to define a type by enumerating its possible variants.

Example:
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // Method body would be defined here
        println!("Message called");
    }
}

fn main() {
    let m = Message::Write(String::from("hello"));
    m.call();
}
```

## 13. Pattern Matching with Option and Result Enum

### Option<T>

`Option<T>` represents an optional value: every `Option` is either `Some` and contains a value, or `None`, and does not.

Example:
```rust
fn divide(numerator: f64, denominator: f64) -> Option<f64> {
    if denominator == 0.0 {
        None
    } else {
        Some(numerator / denominator)
    }
}

fn main() {
    let result = divide(10.0, 2.0);
    match result {
        Some(x) => println!("Result: {}", x),
        None => println!("Cannot divide by zero"),
    }

    // Using if let
    if let Some(x) = divide(10.0, 0.0) {
        println!("Result: {}", x);
    } else {
        println!("Division by zero is not allowed");
    }
}
```

### Result<T, E>

`Result<T, E>` is used for returning and propagating errors. It has two variants: `Ok(T)`, representing success and containing a value, and `Err(E)`, representing error and containing an error value.

Example:
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error)
            }
        },
    };
}
```

## 14-15. Error Handling

Rust groups errors into two major categories: recoverable and unrecoverable errors.

### Unrecoverable Errors with panic!

For unrecoverable errors, the `panic!` macro is used.

Example:
```rust
fn main() {
    panic!("crash and burn");
}
```

### Recoverable Errors with Result

For recoverable errors, the `Result` enum is used.

Example:
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

Using `unwrap` and `expect`:

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

## 16. Generics

Generics allow you to write code that works with multiple types. Here's a practical example:

```rust
use std::ops::Add;

struct Point<T> {
    x: T,
    y: T,
}

impl<T: Add<Output = T>> Add for Point<T> {
    type Output = Self;

    fn add(self, other: Self) -> Self::Output {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    let p3 = p1 + p2;

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);

    let p4 = Point { x: 1.1, y: 1.2 };
    let p5 = Point { x: 2.1, y: 2.2 };
    let p6 = p4 + p5;

    println!("p6.x = {}, p6.y = {}", p6.x, p6.y);
}
```

This example demonstrates:
- A generic `Point` struct that can work with any type `T`.
- Implementation of the `Add` trait for `Point<T>`, allowing points to be added together.
- The use of the same struct and implementation for both integer and floating-point types.

## 17. Traits

Traits define shared behavior across types. They're similar to interfaces in other languages.

Example:
```rust
trait Summary {
    fn summarize(&self) -> String;
}

struct NewsArticle {
    headline: String,
    location: String,
    author: String,
    content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

struct Tweet {
    username: String,
    content: String,
    reply: bool,
    retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```

## 18. Lifetimes

Lifetimes are Rust's way of ensuring that references are valid for as long as we need them to be. Every reference in Rust has a lifetime, which is the scope for which that reference is valid.

Key points:
1. Lifetimes are usually implicit and inferred by the compiler.
2. We must annotate lifetimes when the lifetimes of references could be related in a few different ways.
3. Lifetime annotations don't change how long any of the references live. They describe the relationships of the lifetimes of multiple references to each other.

Example with explanation:

```rust
// 'a is a lifetime parameter
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
        println!("Longest string: {}", result);
    }
}
```

In this example:
- `'a` is a lifetime parameter that represents the scope for which the returned reference is valid.
- The function signature `longest<'a>(x: &'a str, y: &'a str) -> &'a str` tells the compiler that the returned reference will be valid for the smaller of the lifetimes of `x` and `y`.
- This prevents us from using the result after the shorter-lived input reference has gone out of scope.

Lifetimes in struct definitions:

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

This struct definition means an instance of `ImportantExcerpt` can't outlive the reference it holds in its `part` field.

## 19. Cargo

Cargo is Rust's package manager and build system. It handles many tasks:

1. Creating new projects: `cargo new project_name`
2. Building projects: `cargo build`
3. Running projects: `cargo run`
4. Testing: `cargo test`
5. Checking code without producing an executable: `cargo check`
6. Building documentation: `cargo doc`
7. Publishing libraries to crates.io: `cargo publish`

Cargo.toml file:
```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
```

Adding a dependency:
```
cargo add some_crate
```

Updating dependencies:
```
cargo update
```

Cargo workspaces allow you to manage multiple related packages:

```toml
[workspace]
members = [
    "package1",
    "package2",
]
```
