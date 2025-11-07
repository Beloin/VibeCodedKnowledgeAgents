# Rust Ownership and Borrowing - Comprehensive Guide

## Introduction to Ownership

Ownership is Rust's most unique feature that enables memory safety without garbage collection. It's a set of rules that the compiler checks at compile time to ensure memory safety.

### The Three Ownership Rules

1. **Each value has exactly one owner**
   - Every piece of data in Rust has exactly one variable that owns it
   - When the owner goes out of scope, the value is dropped

2. **Only one owner at a time**
   - Ownership can be transferred (moved), but only one variable can own a value at any moment
   - This prevents multiple variables from trying to free the same memory

3. **Values are dropped when owners go out of scope**
   - Rust automatically calls the `drop` function when a value goes out of scope
   - This ensures proper cleanup of resources

## Stack vs Heap Memory

### Stack Memory
- **Characteristics**:
  - Stores values with known, fixed sizes at compile time
  - Very fast allocation and deallocation (LIFO)
  - Automatic memory management
  - Limited in size

- **Examples of stack-allocated types**:
  - Primitive types (`i32`, `f64`, `bool`, `char`)
  - Tuples and arrays with fixed sizes
  - Structs with fixed-size fields

```rust
// Stack allocation examples
let x = 5;           // i32 on stack
let y = true;        // bool on stack
let tuple = (1, 2);  // tuple on stack
let array = [1, 2, 3]; // array on stack
```

### Heap Memory
- **Characteristics**:
  - Stores values with unknown or changing sizes
  - Slower allocation and deallocation
  - Manual allocation required (via `Box`, `Vec`, `String`, etc.)
  - Larger capacity

- **Examples of heap-allocated types**:
  - `String`
  - `Vec<T>`
  - `Box<T>`
  - Any dynamically sized type

```rust
// Heap allocation examples
let s = String::from("hello");  // String on heap
let v = vec![1, 2, 3];          // Vec on heap
let b = Box::new(5);            // Boxed i32 on heap
```

## Move Semantics in Depth

### What Happens During a Move

When a value is moved, Rust:
1. Transfers ownership from the source to the destination
2. Invalidates the source variable
3. Ensures the value is only dropped once

```rust
fn demonstrate_moves() {
    let s1 = String::from("hello");  // s1 owns the String
    
    // Move s1 to s2
    let s2 = s1;                    // s1 is now invalid
    
    // This would cause a compile error:
    // println!("{}", s1);          // Error: borrow of moved value
    
    println!("{}", s2);            // This works fine
} // s2 goes out of scope, String is dropped
```

### Types That Implement Copy

Some types implement the `Copy` trait, which means they are copied instead of moved:

```rust
// Copy types (stack-only)
let x = 5;
let y = x;        // Copy occurs, both x and y are valid
println!("x: {}, y: {}", x, y);  // Both work

// Non-Copy types (heap-involved)
let s1 = String::from("hello");
let s2 = s1;      // Move occurs, s1 is invalidated
// println!("{}", s1);  // Error!
```

**Common Copy types**:
- All integer types (`i32`, `u64`, etc.)
- Floating-point types (`f32`, `f64`)
- `bool`
- `char`
- Tuples containing only Copy types
- Arrays containing only Copy types

## Borrowing and References

### Immutable References (`&`)

Immutable references allow reading data without taking ownership:

```rust
fn calculate_length(s: &String) -> usize {
    s.len()  // We can read but not modify
}

fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
    // s1 is still valid here
}
```

### Mutable References (`&mut`)

Mutable references allow modifying data without taking ownership:

```rust
fn change(s: &mut String) {
    s.push_str(", world");
}

fn main() {
    let mut s = String::from("hello");
    change(&mut s);
    println!("{}", s);  // Prints "hello, world"
}
```

## The Borrow Checker Rules

### Rule 1: Either One Mutable Reference OR Multiple Immutable References

```rust
fn borrowing_rules() {
    let mut data = String::from("hello");
    
    // Multiple immutable references - OK
    let r1 = &data;
    let r2 = &data;
    println!("{} and {}", r1, r2);
    
    // One mutable reference - OK
    let r3 = &mut data;
    r3.push_str(", world");
    
    // This would cause an error:
    // let r4 = &data;        // Error: cannot borrow as immutable
    // println!("{}", r4);    // while mutable reference exists
}
```

### Rule 2: References Must Always Be Valid

```rust
fn dangling_reference_example() {
    let reference;
    {
        let s = String::from("hello");
        // reference = &s;  // Error: s doesn't live long enough
    } // s goes out of scope here
    // println!("{}", reference);  // Would be a dangling reference
}

fn valid_reference_example() {
    let s = String::from("hello");
    let reference = &s;  // OK: s lives longer than reference
    println!("{}", reference);
}
```

## Practical Patterns and Best Practices

### Function Parameter Patterns

```rust
// Prefer borrowing over taking ownership when possible
fn process_string(s: &str) -> usize {
    s.len()
}

// Only take ownership when you need to consume the value
fn consume_string(s: String) -> String {
    s.to_uppercase()
}

// Use mutable references for in-place modification
fn append_text(s: &mut String, text: &str) {
    s.push_str(text);
}
```

### Struct Design Patterns

```rust
struct User {
    username: String,
    email: String,
    active: bool,
}

impl User {
    // Constructor that takes ownership
    fn new(username: String, email: String) -> Self {
        Self {
            username,
            email,
            active: true,
        }
    }
    
    // Methods that borrow immutably
    fn username(&self) -> &str {
        &self.username
    }
    
    // Methods that borrow mutably
    fn deactivate(&mut self) {
        self.active = false;
    }
    
    // Methods that take ownership (consuming)
    fn into_username(self) -> String {
        self.username
    }
}
```

## Common Pitfalls and Solutions

### Pitfall 1: Moving Borrowed Values

```rust
fn problematic_function() {
    let mut data = vec![1, 2, 3];
    let first = &data[0];  // Immutable borrow
    
    // data.push(4);       // Error: cannot borrow as mutable
    // while immutable reference exists
    
    println!("{}", first);  // Use the reference first
    data.push(4);           // Now it's OK
}
```

### Pitfall 2: Iterator Invalidation

```rust
fn safe_iteration() {
    let mut numbers = vec![1, 2, 3, 4, 5];
    
    // Safe: iterate using indices
    for i in 0..numbers.len() {
        if numbers[i] % 2 == 0 {
            numbers[i] *= 2;
        }
    }
    
    // Alternative: use iter_mut()
    for num in numbers.iter_mut() {
        if *num % 2 == 0 {
            *num *= 2;
        }
    }
}
```

### Pitfall 3: Returning References

```rust
// Problematic: returning reference to local variable
// fn get_string() -> &String {
//     let s = String::from("hello");
//     &s  // Error: s doesn't live long enough
// }

// Solution 1: Return owned value
fn get_string_owned() -> String {
    String::from("hello")
}

// Solution 2: Use static lifetime
fn get_static_str() -> &'static str {
    "hello"
}
```

## Advanced Ownership Patterns

### Interior Mutability with RefCell

```rust
use std::cell::RefCell;

struct Cache {
    data: RefCell<Vec<String>>,
}

impl Cache {
    fn new() -> Self {
        Self {
            data: RefCell::new(Vec::new()),
        }
    }
    
    fn add(&self, item: String) {
        self.data.borrow_mut().push(item);
    }
    
    fn get(&self, index: usize) -> Option<String> {
        self.data.borrow().get(index).cloned()
    }
}
```

### Multiple Ownership with Rc

```rust
use std::rc::Rc;

struct Node {
    value: i32,
    next: Option<Rc<Node>>,
}

fn create_linked_list() {
    let node3 = Rc::new(Node { value: 3, next: None });
    let node2 = Rc::new(Node { value: 2, next: Some(Rc::clone(&node3)) });
    let node1 = Rc::new(Node { value: 1, next: Some(Rc::clone(&node2)) });
    
    // Multiple references to the same node
    let another_ref = Rc::clone(&node2);
}
```

## Real-World Applications

### Building a Parser

```rust
struct Parser<'a> {
    input: &'a str,
    position: usize,
}

impl<'a> Parser<'a> {
    fn new(input: &'a str) -> Self {
        Self { input, position: 0 }
    }
    
    fn parse_identifier(&mut self) -> Option<&'a str> {
        let start = self.position;
        while self.position < self.input.len() {
            let c = self.input.chars().nth(self.position).unwrap();
            if c.is_alphabetic() {
                self.position += 1;
            } else {
                break;
            }
        }
        
        if start == self.position {
            None
        } else {
            Some(&self.input[start..self.position])
        }
    }
}
```

### Database Connection Pool

```rust
use std::sync::{Arc, Mutex};

struct ConnectionPool {
    connections: Arc<Mutex<Vec<Connection>>>,
}

impl ConnectionPool {
    fn get_connection(&self) -> Option<PooledConnection> {
        let mut connections = self.connections.lock().unwrap();
        connections.pop().map(|conn| PooledConnection {
            conn,
            pool: Arc::clone(&self.connections),
        })
    }
}

struct PooledConnection {
    conn: Connection,
    pool: Arc<Mutex<Vec<Connection>>>,
}

impl Drop for PooledConnection {
    fn drop(&mut self) {
        let mut pool = self.pool.lock().unwrap();
        pool.push(std::mem::replace(&mut self.conn, Connection::new()));
    }
}
```

## Performance Considerations

- **Prefer borrowing over cloning**: Avoid unnecessary memory allocations
- **Use slices (`&[T]`) instead of `Vec<T>`**: When you only need to read data
- **Consider `Cow` (Clone-on-Write)**: For scenarios where you might need to modify data but want to avoid cloning if possible
- **Use `Box` for large types**: To avoid stack overflow with large structs

## Summary

Rust's ownership system provides:
- **Memory safety** without garbage collection
- **Zero-cost abstractions** for memory management
- **Compile-time guarantees** against common bugs
- **Clear API design** through ownership semantics

Mastering ownership and borrowing is essential for writing efficient, safe Rust code. The compiler's strict checks ensure that once your code compiles, it's free from many common memory-related bugs.