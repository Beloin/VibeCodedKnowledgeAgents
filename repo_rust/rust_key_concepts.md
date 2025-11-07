# Rust Programming Language - Key Concepts

## Overview

Rust is a systems programming language that focuses on safety, speed, and concurrency. It achieves memory safety without garbage collection through its unique ownership system and compile-time checks.

## 1. Ownership and Borrowing

### Ownership Rules
- **Each value has an owner**: Every piece of data in Rust has exactly one variable that owns it
- **Only one owner at a time**: Ownership can be transferred, but only one variable can own a value at any given moment
- **Values are dropped when owners go out of scope**: When the owner goes out of scope, Rust automatically calls `drop` to clean up the memory

### Stack vs Heap
- **Stack**: Stores values with known, fixed sizes. Fast access, automatic allocation/deallocation
- **Heap**: Stores values with unknown or changing sizes. Slower access, manual allocation required

### Move Semantics
```rust
let s1 = String::from("hello");
let s2 = s1;  // s1 is moved to s2, s1 is no longer valid
// println!("{}", s1);  // This would cause a compile error
```

### Borrowing and References
- **Immutable references (`&`)**: Allow reading but not modifying
- **Mutable references (`&mut`)**: Allow reading and modifying
- **Borrowing rules**:
  - You can have either one mutable reference OR any number of immutable references
  - References must always be valid

```rust
fn calculate_length(s: &String) -> usize {
    s.len()
}

fn change(s: &mut String) {
    s.push_str(", world");
}
```

## 2. Memory Safety Features

### No Null Pointers
- Rust doesn't have null pointers
- Uses `Option<T>` enum to represent optional values

### No Dangling References
- Compiler ensures references never point to invalid memory
- Prevents use-after-free and double-free errors

### Compile-Time Memory Safety
- Ownership system prevents memory leaks
- Borrow checker ensures references are valid
- No data races in concurrent code

## 3. Type System and Traits

### Strong Static Typing
- All types are known at compile time
- Type inference reduces verbosity

### Generics
- Allow writing code that works with multiple types
- Zero-cost abstractions

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}
```

### Traits
- Define shared behavior across types
- Similar to interfaces in other languages
- Can be used for trait bounds in generics

```rust
trait Summary {
    fn summarize(&self) -> String;
}

struct NewsArticle {
    headline: String,
    location: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, from {}", self.headline, self.location)
    }
}
```

### Common Traits
- `Copy`: Types that can be copied bit-for-bit
- `Clone`: Types that can be explicitly cloned
- `Drop`: Custom cleanup when values go out of scope
- `Debug`: Formatting for debugging
- `Display`: User-facing formatting

## 4. Structs and Methods

### Struct Definition
```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

### Method Implementation
```rust
impl User {
    fn new(username: String, email: String) -> Self {
        Self {
            username,
            email,
            sign_in_count: 0,
            active: true,
        }
    }
    
    fn deactivate(&mut self) {
        self.active = false;
    }
}
```

### Associated Functions
```rust
impl User {
    fn anonymous() -> Self {
        Self::new("anonymous".to_string(), "anonymous@example.com".to_string())
    }
}
```

## 5. Enums and Pattern Matching

### Enum Definition
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

### Pattern Matching
```rust
fn process_message(msg: Message) {
    match msg {
        Message::Quit => println!("Quitting"),
        Message::Move { x, y } => println!("Moving to ({}, {})", x, y),
        Message::Write(text) => println!("Writing: {}", text),
        Message::ChangeColor(r, g, b) => println!("Changing color to RGB({}, {}, {})", r, g, b),
    }
}
```

### Option and Result Enums
```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

## 6. Error Handling

### Two Kinds of Errors
- **Recoverable errors**: Use `Result<T, E>` type
- **Unrecoverable errors**: Use `panic!` macro

### Result Type
```rust
fn read_file(path: &str) -> Result<String, io::Error> {
    fs::read_to_string(path)
}
```

### Error Propagation
- Use `?` operator for concise error handling
- Automatically converts error types when possible

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut file = File::open("username.txt")?;
    let mut username = String::new();
    file.read_to_string(&mut username)?;
    Ok(username)
}
```

## 7. Modules and Visibility

### Module System
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
        fn seat_at_table() {}
    }
    
    mod serving {
        fn take_order() {}
    }
}

pub fn eat_at_restaurant() {
    front_of_house::hosting::add_to_waitlist();
}
```

### Visibility Modifiers
- `pub`: Public visibility
- No modifier: Private (default)
- `pub(crate)`: Visible within current crate
- `pub(super)`: Visible to parent module

### Use Declarations
```rust
use std::collections::HashMap;
use std::fmt::Result;
use std::io::Result as IoResult;
```

## 8. Smart Pointers

### Box<T>
- Stores data on the heap
- Single ownership
- Fixed size pointer

```rust
let b = Box::new(5);
println!("b = {}", b);
```

### Rc<T>
- Reference counting pointer
- Multiple ownership
- Non-thread-safe

```rust
use std::rc::Rc;

let a = Rc::new(String::from("hello"));
let b = Rc::clone(&a);
```

### Arc<T>
- Atomic reference counting pointer
- Thread-safe version of Rc

```rust
use std::sync::Arc;
use std::thread;

let counter = Arc::new(0);
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        // Use counter
    });
    handles.push(handle);
}
```

### RefCell<T>
- Interior mutability pattern
- Runtime borrow checking

```rust
use std::cell::RefCell;

let x = RefCell::new(42);
{
    let mut y = x.borrow_mut();
    *y += 1;
}
println!("{}", x.borrow());
```

## 9. Closures

### Closure Syntax
```rust
let add_one = |x: i32| -> i32 { x + 1 };
let add_one_short = |x| x + 1;
```

### Capturing Environment
```rust
let x = 4;
let equal_to_x = |z| z == x;
println!("{} equal to x: {}", 4, equal_to_x(4));
```

### Function Traits
- `Fn`: Immutable borrow of captured variables
- `FnMut`: Mutable borrow of captured variables
- `FnOnce`: Takes ownership of captured variables

## 10. Iterators

### Iterator Pattern
```rust
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();

for val in v1_iter {
    println!("Got: {}", val);
}
```

### Common Iterator Methods
```rust
let numbers = vec![1, 2, 3, 4, 5];

// Transform and collect
let doubled: Vec<i32> = numbers.iter().map(|x| x * 2).collect();

// Filter
let evens: Vec<&i32> = numbers.iter().filter(|x| *x % 2 == 0).collect();

// Fold/Reduce
let sum: i32 = numbers.iter().fold(0, |acc, x| acc + x);

// Chain operations
let result: Vec<i32> = numbers
    .iter()
    .filter(|x| *x > &2)
    .map(|x| x * 3)
    .collect();
```

### Creating Iterators
```rust
// From ranges
let range_iter = (1..10).into_iter();

// From custom types
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.count < 5 {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}
```

## 11. Concurrency Model

### Fearless Concurrency
- Compile-time prevention of data races
- Ownership system extends to concurrent programming

### Threads
- Create threads with `std::thread::spawn`
- Join handles to wait for thread completion

```rust
use std::thread;

let handle = thread::spawn(|| {
    // thread code
});

handle.join().unwrap();
```

### Message Passing
- Use channels to send messages between threads
- Multiple producer, single consumer (mpsc)

```rust
use std::sync::mpsc;

let (tx, rx) = mpsc::channel();

tx.send("hello").unwrap();
let received = rx.recv().unwrap();
```

### Shared State
- Use `Mutex` and `Arc` for shared mutable state
- `Mutex` provides mutual exclusion
- `Arc` enables atomic reference counting

```rust
use std::sync::{Arc, Mutex};

let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    });
    handles.push(handle);
}
```

### Sync and Send Traits
- `Send`: Types that can be transferred between threads
- `Sync`: Types that can be referenced from multiple threads

## 12. Testing Framework

### Unit Tests
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
    
    #[test]
    #[should_panic]
    fn it_fails() {
        panic!("This test should fail");
    }
}
```

### Integration Tests
```rust
// tests/integration_test.rs
use my_crate;

#[test]
fn test_public_api() {
    assert_eq!(my_crate::add(2, 2), 4);
}
```

### Test Organization
- Unit tests: In same file as code being tested
- Integration tests: In `tests/` directory
- Documentation tests: In code comments

### Common Testing Macros
- `assert!`: Check if boolean is true
- `assert_eq!`: Check if two values are equal
- `assert_ne!`: Check if two values are not equal
- `#[should_panic]`: Test should panic
- `#[ignore]`: Ignore test temporarily

## 13. Package Management and Cargo

### Cargo Features
- **Build system**: Compiles code and dependencies
- **Package manager**: Downloads and manages dependencies
- **Test runner**: Runs tests and benchmarks
- **Documentation generator**: Creates documentation

### Project Structure
```
project/
├── Cargo.toml
├── Cargo.lock
└── src/
    ├── main.rs
    ├── lib.rs
    └── bin/
```

### Cargo.toml
```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = "1.0"
reqwest = { version = "0.11", features = ["json"] }

[dev-dependencies]
assert_eq = "1.0"
```

### Common Cargo Commands
- `cargo new`: Create new project
- `cargo build`: Compile project
- `cargo run`: Compile and run
- `cargo check`: Check compilation without building
- `cargo test`: Run tests
- `cargo doc`: Generate documentation
- `cargo publish`: Publish to crates.io

## 14. Advanced Features

### Lifetimes
- Ensure references are valid for required duration
- Annotated with `'a`, `'static`, etc.

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### Macros
- `macro_rules!` for declarative macros
- Procedural macros for more complex code generation

### Async/Await
- Built-in support for asynchronous programming
- Zero-cost abstractions for async I/O

```rust
async fn fetch_data() -> Result<String, reqwest::Error> {
    reqwest::get("https://example.com")
        .await?
        .text()
        .await
}
```

## Resources for Further Learning

### Official Documentation
- [The Rust Programming Language Book](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Rust Standard Library Documentation](https://doc.rust-lang.org/std/)

### Community Resources
- [Rustlings](https://github.com/rust-lang/rustlings) - Small exercises
- [Exercism Rust Track](https://exercism.org/tracks/rust)
- [The Rustonomicon](https://doc.rust-lang.org/nomicon/) - Advanced unsafe Rust

### Tools
- [rustup](https://rustup.rs/) - Rust toolchain installer
- [cargo](https://doc.rust-lang.org/cargo/) - Package manager
- [clippy](https://github.com/rust-lang/rust-clippy) - Linter
- [rustfmt](https://github.com/rust-lang/rustfmt) - Code formatter

---

*This document was created based on the official Rust documentation and serves as a comprehensive guide to Rust's key concepts for learners and developers.*