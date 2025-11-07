# Rust Advanced Features - Comprehensive Guide

## Overview

Rust provides several advanced features that enable sophisticated programming patterns while maintaining safety and performance. These features include lifetimes, macros, async/await, unsafe code, and more, allowing developers to write high-performance systems code with fine-grained control.

## Lifetimes

### What Are Lifetimes?

Lifetimes are Rust's way of ensuring that references are always valid. They're annotations that tell the compiler how long references should live, preventing dangling references and use-after-free errors.

### Basic Lifetime Syntax

```rust
// Function with explicit lifetime annotations
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";
    
    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

### Lifetime Elision Rules

Rust has three lifetime elision rules that allow omitting explicit annotations in common cases:

```rust
// Rule 1: Each parameter gets its own lifetime
fn first_rule(s: &str) -> &str;  // Becomes: fn first_rule<'a>(s: &'a str) -> &'a str

// Rule 2: If there's exactly one input lifetime, it's assigned to all output lifetimes
fn second_rule(s: &str) -> &str;  // Becomes: fn second_rule<'a>(s: &'a str) -> &'a str

// Rule 3: If there are multiple input lifetimes but one is &self or &mut self,
// the lifetime of self is assigned to all output lifetimes
impl SomeStruct {
    fn third_rule(&self, s: &str) -> &str;  // Becomes: fn third_rule<'a, 'b>(&'a self, s: &'b str) -> &'a str
}
```

### Structs with Lifetimes

```rust
// Struct holding a reference
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
    
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
    
    println!("Excerpt: {}", i.part);
}
```

### Static Lifetime

```rust
// 'static lifetime lasts for the entire program duration
let s: &'static str = "I have a static lifetime";

// Common in error messages
fn create_error_message() -> &'static str {
    "An error occurred"
}
```

### Advanced Lifetime Patterns

```rust
// Multiple lifetimes
fn multiple_lifetimes<'a, 'b>(x: &'a str, y: &'b str) -> &'a str {
    println!("Second string: {}", y);
    x
}

// Lifetime bounds
struct TwoRefs<'a, 'b: 'a> {
    first: &'a str,
    second: &'b str,  // 'b must outlive 'a
}

// Higher-ranked trait bounds (HRTB)
fn call_with_ref<F>(f: F) 
where 
    F: for<'a> Fn(&'a i32),
{
    let value = 10;
    f(&value);
}
```

## Macros

### Declarative Macros with macro_rules!

```rust
// Simple macro
macro_rules! say_hello {
    () => {
        println!("Hello!");
    };
}

// Macro with parameters
macro_rules! create_function {
    ($func_name:ident) => {
        fn $func_name() {
            println!("You called {:?}()", stringify!($func_name));
        }
    };
}

// Create functions using macro
create_function!(foo);
create_function!(bar);

// Macro with multiple patterns
macro_rules! test {
    ($left:expr; and $right:expr) => {
        println!("{:?} and {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left && $right);
    };
    ($left:expr; or $right:expr) => {
        println!("{:?} or {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left || $right);
    };
}

fn main() {
    say_hello!();
    foo();
    bar();
    test!(1i32 + 1 == 2i32; and 2i32 * 2 == 4i32);
    test!(true; or false);
}
```

### Advanced Declarative Macros

```rust
// Vector creation macro
macro_rules! vector {
    ($($x:expr),*) => {
        {
            let mut temp_vec = Vec::new();
            $(temp_vec.push($x);)*
            temp_vec
        }
    };
}

// HTML builder macro
macro_rules! html {
    (<$tag:ident>$($content:tt)*</$tag:ident>) => {
        format!("<{}>{}</{}>", stringify!($tag), html!($($content)*), stringify!($tag))
    };
    ($text:expr) => { $text };
    () => { "" };
}

fn main() {
    let v = vector![1, 2, 3];
    println!("Vector: {:?}", v);
    
    let html_output = html!(<div>Hello <strong>world</strong></div>);
    println!("HTML: {}", html_output);
}
```

### Procedural Macros

Procedural macros allow creating custom derives, function-like macros, and attribute macros.

#### Custom Derive Macro

```rust
// In Cargo.toml
// [lib]
// proc-macro = true

use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    let ast = syn::parse(input).unwrap();
    impl_hello_macro(&ast)
}

fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    gen.into()
}

// Usage
use hello_macro::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
}
```

#### Attribute-like Macros

```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
    // Process attributes and generate code
    item
}

// Usage
#[route(GET, "/")]
fn index() -> &'static str {
    "Hello, World!"
}
```

#### Function-like Macros

```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    // Parse SQL and generate safe Rust code
    input
}

// Usage
let query = sql!(SELECT * FROM posts WHERE id = 10);
```

## Async/Await

### Basic Async/Await

```rust
use tokio::time::{sleep, Duration};

async fn say_hello() {
    println!("Hello!");
    sleep(Duration::from_secs(1)).await;
    println!("World!");
}

async fn compute() -> i32 {
    sleep(Duration::from_secs(2)).await;
    42
}

#[tokio::main]
async fn main() {
    say_hello().await;
    let result = compute().await;
    println!("Result: {}", result);
}
```

### Concurrent Execution

```rust
use tokio::task;
use tokio::time::{sleep, Duration};

async fn download_file(id: u32) -> String {
    sleep(Duration::from_millis(100 * id)).await;
    format!("File {}", id)
}

#[tokio::main]
async fn main() {
    // Sequential execution
    let file1 = download_file(1).await;
    let file2 = download_file(2).await;
    println!("Sequential: {}, {}", file1, file2);
    
    // Concurrent execution
    let task1 = task::spawn(download_file(3));
    let task2 = task::spawn(download_file(4));
    
    let (file3, file4) = tokio::join!(task1, task2);
    println!("Concurrent: {}, {}", file3.unwrap(), file4.unwrap());
    
    // Using select! for first completion
    tokio::select! {
        file = download_file(5) => println!("First: {}", file),
        file = download_file(6) => println!("First: {}", file),
    }
}
```

### Async Traits

```rust
use async_trait::async_trait;

#[async_trait]
pub trait Database {
    async fn get_user(&self, id: u32) -> Result<String, String>;
    async fn save_user(&self, user: &str) -> Result<(), String>;
}

struct MockDatabase;

#[async_trait]
impl Database for MockDatabase {
    async fn get_user(&self, id: u32) -> Result<String, String> {
        Ok(format!("User {}", id))
    }
    
    async fn save_user(&self, user: &str) -> Result<(), String> {
        println!("Saving user: {}", user);
        Ok(())
    }
}

#[tokio::main]
async fn main() {
    let db = MockDatabase;
    let user = db.get_user(42).await.unwrap();
    println!("User: {}", user);
}
```

### Streams

```rust
use tokio_stream::{self as stream, StreamExt};
use tokio::time::{sleep, Duration};

async fn number_stream() -> impl stream::Stream<Item = i32> {
    stream::iter(1..=5)
}

async fn delayed_stream() -> impl stream::Stream<Item = i32> {
    stream::unfold(1, |state| async move {
        sleep(Duration::from_millis(100)).await;
        if state <= 5 {
            Some((state, state + 1))
        } else {
            None
        }
    })
}

#[tokio::main]
async fn main() {
    // Process stream
    let mut stream = number_stream().await;
    while let Some(number) = stream.next().await {
        println!("Number: {}", number);
    }
    
    // Process delayed stream
    let mut delayed = delayed_stream().await;
    while let Some(number) = delayed.next().await {
        println!("Delayed: {}", number);
    }
}
```

## Unsafe Rust

### Unsafe Superpowers

Unsafe Rust allows you to:
1. Dereference raw pointers
2. Call unsafe functions
3. Implement unsafe traits
4. Access or modify mutable static variables
5. Access union fields

### Raw Pointers

```rust
fn main() {
    let mut num = 5;
    
    // Create raw pointers
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
    
    unsafe {
        // Dereference raw pointers
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
        
        // Modify through mutable raw pointer
        *r2 = 10;
        println!("After modification: {}", *r1);
    }
}
```

### Calling Unsafe Functions

```rust
unsafe fn dangerous() {
    println!("Doing something dangerous!");
}

// Safe wrapper around unsafe function
fn safe_wrapper() {
    unsafe {
        dangerous();
    }
}

// External function (from C)
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    safe_wrapper();
    
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

### Implementing Unsafe Traits

```rust
unsafe trait UnsafeTrait {
    // Some methods
}

struct MyType;

unsafe impl UnsafeTrait for MyType {
    // Implementation
}
```

### Accessing Static Variables

```rust
static mut COUNTER: u32 = 0;

fn add_to_counter(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_counter(3);
    
    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

## Advanced Type System Features

### Associated Types

```rust
trait Iterator {
    type Item;  // Associated type
    
    fn next(&mut self) -> Option<Self::Item>;
}

struct Counter {
    count: u32,
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

### Generic Associated Types (GATs)

```rust
trait StreamingIterator {
    type Item<'a> where Self: 'a;
    
    fn next<'a>(&'a mut self) -> Option<Self::Item<'a>>;
}

struct Windows<'a, T> {
    slice: &'a [T],
    size: usize,
}

impl<'a, T> StreamingIterator for Windows<'a, T> {
    type Item<'b> = &'b [T] where 'a: 'b;
    
    fn next<'b>(&'b mut self) -> Option<Self::Item<'b>> {
        if self.size <= self.slice.len() {
            let window = &self.slice[..self.size];
            self.slice = &self.slice[1..];
            Some(window)
        } else {
            None
        }
    }
}
```

### Higher-Rank Trait Bounds (HRTB)

```rust
// Function that works with any lifetime
fn call_on_ref_zero<F>(f: F) 
where 
    F: for<'a> Fn(&'a i32),
{
    let zero = 0;
    f(&zero);
}

// Trait with HRTB
trait AnyRef {
    fn any_ref(&self, f: impl for<'a> Fn(&'a Self));
}

impl<T> AnyRef for T {
    fn any_ref(&self, f: impl for<'a> Fn(&'a Self)) {
        f(self);
    }
}
```

### Const Generics

```rust
struct Array<T, const N: usize> {
    data: [T; N],
}

impl<T, const N: usize> Array<T, N> {
    fn new() -> Self 
    where 
        T: Default + Copy,
    {
        Self {
            data: [T::default(); N],
        }
    }
    
    fn len(&self) -> usize {
        N
    }
}

// Using const generics
fn main() {
    let array: Array<i32, 10> = Array::new();
    println!("Array length: {}", array.len());
}
```

## Advanced Pattern Matching

### if let and while let

```rust
fn main() {
    let some_value = Some(5);
    
    // if let for matching single patterns
    if let Some(x) = some_value {
        println!("Value: {}", x);
    }
    
    // while let for looping until pattern fails
    let mut stack = Vec::new();
    stack.push(1);
    stack.push(2);
    stack.push(3);
    
    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
}
```

### Pattern Guards

```rust
fn main() {
    let num = Some(4);
    
    match num {
        Some(x) if x < 5 => println!("Less than 5: {}", x),
        Some(x) => println!("{}", x),
        None => (),
    }
    
    // Complex pattern with guards
    let pair = (2, -2);
    
    match pair {
        (x, y) if x == y => println!("These are twins"),
        (x, y) if x + y == 0 => println!("Antimatter, kaboom!"),
        (x, _) if x % 2 == 1 => println!("The first one is odd"),
        _ => println!("No correlation..."),
    }
}
```

### @ Bindings

```rust
fn main() {
    let msg = Message::Hello { id: 5 };
    
    match msg {
        Message::Hello { 
            id: id_variable @ 3..=7,
        } => {
            println!("Found an id in range: {}", id_variable)
        }
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        }
        Message::Hello { id } => {
            println!("Found some other id: {}", id)
        }
    }
}

enum Message {
    Hello { id: i32 },
}
```

## Advanced Error Handling

### Custom Error Types

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum MyError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    
    #[error("Parse error: {0}")]
    Parse(#[from] std::num::ParseIntError),
    
    #[error("Custom error: {message}")]
    Custom { message: String },
    
    #[error("Network error: {status}")]
    Network { status: u16 },
}

fn might_fail() -> Result<(), MyError> {
    // Various operations that might fail
    let _file = std::fs::File::open("nonexistent.txt")?;
    let _number = "not a number".parse::<i32>()?;
    
    if true {
        return Err(MyError::Custom { 
            message: "Something went wrong".to_string() 
        });
    }
    
    Ok(())
}
```

### Error Conversion

```rust
use std::error::Error;
use std::fmt;

#[derive(Debug)]
struct AppError {
    details: String,
}

impl AppError {
    fn new(msg: &str) -> AppError {
        AppError { details: msg.to_string() }
    }
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.details)
    }
}

impl Error for AppError {
    fn description(&self) -> &str {
        &self.details
    }
}

// Convert from other error types
impl From<std::io::Error> for AppError {
    fn from(err: std::io::Error) -> Self {
        AppError::new(&format!("IO error: {}", err))
    }
}
```

## Conclusion

Rust's advanced features provide powerful tools for systems programming while maintaining safety guarantees. Lifetimes ensure memory safety, macros enable metaprogramming, async/await facilitates concurrent programming, and unsafe code allows low-level operations when necessary. These features, combined with Rust's strong type system, make it possible to write high-performance, reliable systems software.