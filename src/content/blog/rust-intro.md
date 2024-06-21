---
title: "Introduction to Rust Programming Language"
description: "This blog introduces rust programming language and who am I, how to install it and create new projects with rust"
pubDate: "June 21, 2024"
heroImage: "/rust.jpg"
tags: ["RustSeries"]
---
# Introduction

Hey Everyone, welcome to my rust programming series. In this series, we are going to deep dive into the world of rust, in order to learn what is rust programming language and how we can use it in order to create memory safe and blazingly fast software systems.

According to Rust By Example, [Rust](https://www.rust-lang.org/) is a modern systems programming language focusing on safety, speed, and concurrency. It accomplishes these goals by being memory safe without using garbage collection. Rust can be used in writing any kind of software, where ever memory safety and speed is needed.

Rust has been ranked as the most loved programming language for over 7 years by the survey done at Stack Overflow, and has been steadily in rise of number of developers and projects. Rust has been used in big companies such as Microsoft, Mozila, Google, Meta and Apple. Rust has been the go-to-language in terms of building blockchain and modern smart contracts. Modern smart contract DSL such as move, cairo and ink are created as a superset of rust and follows rust heavily.

Even though, Rust is being the most loved language among devs, one major problem with rust is the steep learning curve. Resources such as Rust Book, Rust by Example are very well documented but still very hard to understand for non-native english speakers. This whole series is a adaptation of how I learned Rust and introducing a developer prescriptive to rust.

# Who I am and What I do?

I am Sudharsanan Kirubanandhan, a software developer(5.6 years experience) who worked predominantly in blockchain and AI. At the time of writing this blog, I am jobless. Yeah, you heared it right, my final job was at astra technologies four months ago, where i worked for 7 months in rust backend development. After that, I tried building my a decentralised AI startup and failed. The idea of writing this blog is because I love software development, and my world is completely surrounded around it. I always love to teach the little things I know to the world, because I always thought that's how some one grow. I am planning to write this entire series, while I am relearning rust and I hope this series can help you to get started with rust. If you guys have any job oppurtunities, project oppurtunities , training and bootcamp oppurtunities please write me a email to sudharsanan\@thegeekstrategy.com.

# Installing Rust

I am using a macbook pro to learn rust and write this blog, For windows and linux users, I will be adding the essential links to do the same.
For Macbook users, the recommended way is to use rustup, open the terminal and run the following command

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

and this works for linux users too. For finding the recommened way for installing rust in your system, follow this link https://www.rust-lang.org/tools/install
After Installation, to check if rust is properly installed, hit the following command

```
rustc --version
```

My system currently runs rustc 1.78.0 (9b00956e5 2024-04-29).

once rust is installed, the next thing to check is cargo.

# Cargo Package Manager

For those who are familiar with NPM, you may easily understand what cargo is . Like NPM for Nodejs, cargo is the official package manager for rust.

It has lots of really useful features to improve code quality and developer velocity! These include

* Dependency management and integration with [crates.io](https://crates.io/) (the official Rust package registry)
* Awareness of unit tests
* Awareness of benchmarks

# Hello World

lets start our rust journey, by writting a simple hello world application

```
cargo new helloworld 
```

This will create a new rust project, The project will consists of two folders src and target, and cargo.toml and cargo.lock files
The cargo.toml file is similar to package.json where packages tells about the created packages and any dependecies needed for the particular file to run should be placed under the dependencies section

```
//cargo.toml
[package]
name = "helloworld"
version = "0.1.0"
edition = "2021"

[dependencies]

```

Inside the main folder, you can see the main.rs file, which is as follows

```
fn main() {
    println!("Hello, world!");
}

```

In this program, the execution starts from the main function and we have used a println! macro to print out Hello world. You can run this program with the following command

```
cd helloworld
cargo run
```

In the next blog, we will be taking about the data types and write a program to understand the same. Watch out [sksudharsanan.com](https://sksudharsanan.com) for more blogs and all rust series blogs can be found [here](https://sksudharsanan.com/blog/tag/RustSeries)

Thanks a lot for giving your time in reading this blog and hope this series can help you get started with rust programming language
