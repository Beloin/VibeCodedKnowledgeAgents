# Rust Enums and Pattern Matching - Comprehensive Guide

## Introduction to Enums

Enums (enumerations) allow you to define a type by enumerating its possible variants. They are one of Rust's most powerful features, enabling you to model complex data structures and state machines.

## Basic Enum Definition

### Simple Enum

```rust
enum IpAddrKind {
    V4,
    V6,
}

fn route(ip_kind: IpAddrKind) {
    match ip_kind {
        IpAddrKind::V4 => println!("Routing IPv4"),
        IpAddrKind::V6 => println!("Routing IPv6"),
    }
}

fn basic_enum_example() {
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
    
    route(four);
    route(six);
}
```

### Enums with Data

Enums can hold data of different types:

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),      // Tuple variant
    V6(String),               // Struct-like variant
}

fn enum_with_data_example() {
    let home = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));
    
    match home {
        IpAddr::V4(a, b, c, d) => println!("IPv4: {}.{}.{}.{}", a, b, c, d),
        IpAddr::V6(addr) => println!("IPv6: {}", addr),
    }
}
```

### Complex Enum Variants

```rust
enum Message {
    Quit,                           // No data
    Move { x: i32, y: i32 },       // Named fields
    Write(String),                  // Single value
    ChangeColor(i32, i32, i32),    // Multiple values
}

impl Message {
    fn call(&self) {
        // Method implementation
    }
}

fn complex_enum_example() {
    let messages = vec![
        Message::Quit,
        Message::Move { x: 10, y: 20 },
        Message::Write(String::from("hello")),
        Message::ChangeColor(255, 0, 0),
    ];
    
    for msg in messages {
        process_message(msg);
    }
}

fn process_message(msg: Message) {
    match msg {
        Message::Quit => println!("Quit message"),
        Message::Move { x, y } => println!("Move to ({}, {})", x, y),
        Message::Write(text) => println!("Write: {}", text),
        Message::ChangeColor(r, g, b) => println!("Change color to RGB({}, {}, {})", r, g, b),
    }
}
```

## The Option Enum

### Handling Absence of Values

Rust doesn't have null pointers. Instead, it uses `Option<T>`:

```rust
enum Option<T> {
    Some(T),
    None,
}

fn option_examples() {
    let some_number = Some(5);
    let some_string = Some("a string");
    let absent_number: Option<i32> = None;
    
    // You must handle both cases
    match some_number {
        Some(value) => println!("Got: {}", value),
        None => println!("Got nothing"),
    }
}
```

### Common Option Methods

```rust
fn option_methods_examples() {
    let x: Option<i32> = Some(5);
    let y: Option<i32> = None;
    
    // Unwrapping with defaults
    let value1 = x.unwrap_or(0);        // 5
    let value2 = y.unwrap_or(0);        // 0
    
    // Mapping values
    let doubled = x.map(|v| v * 2);     // Some(10)
    let still_none = y.map(|v| v * 2);  // None
    
    // Filtering
    let filtered = x.filter(|&v| v > 3); // Some(5)
    let filtered_none = x.filter(|&v| v > 10); // None
    
    // Chaining operations
    let result = x
        .map(|v| v + 1)
        .filter(|&v| v % 2 == 0)
        .unwrap_or(0);
    
    // Working with references
    let string_opt: Option<String> = Some("hello".to_string());
    let length_opt = string_opt.as_ref().map(|s| s.len());
    // string_opt is still valid here
}
```

## The Result Enum

### Error Handling

Rust uses `Result<T, E>` for recoverable errors:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

fn result_examples() -> Result<i32, String> {
    let success: Result<i32, String> = Ok(42);
    let failure: Result<i32, String> = Err("Something went wrong".to_string());
    
    match success {
        Ok(value) => println!("Success: {}", value),
        Err(error) => println!("Error: {}", error),
    }
    
    // Using ? operator for error propagation
    let value = success?;
    Ok(value)
}
```

### Common Result Methods

```rust
use std::fs::File;
use std::io::{self, Read};

fn result_methods_examples() -> Result<String, io::Error> {
    let file_result = File::open("example.txt");
    
    // Handling results
    match file_result {
        Ok(file) => {
            // Process file
            Ok(String::new())
        }
        Err(error) => {
            eprintln!("Failed to open file: {}", error);
            Err(error)
        }
    }
    
    // Using unwrap (for prototyping)
    let file = File::open("example.txt").unwrap();
    
    // Using expect for better error messages
    let file = File::open("example.txt").expect("Failed to open example.txt");
    
    // Using map_err for error transformation
    let content = std::fs::read_to_string("example.txt")
        .map_err(|e| format!("File error: {}", e))?;
    
    Ok(content)
}
```

## Pattern Matching

### Basic Match Expressions

```rust
fn match_basics(value: i32) {
    match value {
        1 => println!("One"),
        2 => println!("Two"),
        3 => println!("Three"),
        _ => println!("Something else"),  // Default case
    }
}
```

### Matching on Enums

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### Patterns That Bind to Values

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // ... other states
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),  // Quarter variant holds a UsState value
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

### Matching with Option<T>

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

fn option_matching_example() {
    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
    
    println!("six: {:?}", six);    // Some(6)
    println!("none: {:?}", none);  // None
}
```

### Exhaustive Matching

Rust requires match expressions to be exhaustive:

```rust
fn exhaustive_match(value: Option<i32>) {
    match value {
        Some(x) if x > 0 => println!("Positive: {}", x),
        Some(0) => println!("Zero"),
        Some(x) => println!("Negative: {}", x),
        None => println!("No value"),
    }
}
```

## Advanced Pattern Matching

### Multiple Patterns

```rust
fn multiple_patterns(value: i32) {
    match value {
        1 | 2 | 3 => println!("Small number"),
        4..=6 => println!("Medium number"),
        7..=10 => println!("Large number"),
        _ => println!("Out of range"),
    }
}
```

### Destructuring

```rust
struct Point {
    x: i32,
    y: i32,
}

enum Shape {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
}

fn destructuring_examples() {
    let point = Point { x: 10, y: 20 };
    
    match point {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
    
    let shape = Shape::Rectangle { width: 10.0, height: 20.0 };
    
    match shape {
        Shape::Circle { radius } => println!("Circle with radius: {}", radius),
        Shape::Rectangle { width, height } => {
            println!("Rectangle {}x{}", width, height);
        }
    }
}
```

### Pattern Guards

```rust
fn pattern_guards(num: Option<i32>) {
    match num {
        Some(x) if x < 0 => println!("Negative: {}", x),
        Some(x) if x == 0 => println!("Zero"),
        Some(x) => println!("Positive: {}", x),
        None => println!("No number"),
    }
}

fn complex_guard(point: (i32, i32)) {
    match point {
        (x, y) if x == y => println!("On the diagonal"),
        (x, y) if x == -y => println!("On the anti-diagonal"),
        (x, y) => println!("Somewhere else: ({}, {})", x, y),
    }
}
```

### @ Bindings

```rust
fn at_binding_example() {
    let msg = Message::Move { x: 10, y: 20 };
    
    match msg {
        Message::Move { x: x_val @ 0..=10, y: y_val @ 0..=10 } => {
            println!("Small move: ({}, {})", x_val, y_val);
        }
        Message::Move { x, y } => {
            println!("Large move: ({}, {})", x, y);
        }
        _ => (),
    }
}
```

## if let Syntax

### Concise Pattern Matching

```rust
fn if_let_examples() {
    let some_value = Some(3);
    
    // Instead of:
    match some_value {
        Some(3) => println!("three"),
        _ => (),
    }
    
    // Use if let:
    if let Some(3) = some_value {
        println!("three");
    }
    
    // With else clause
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    } else {
        println!("No maximum configured");
    }
}
```

### while let Loops

```rust
fn while_let_example() {
    let mut stack = Vec::new();
    
    stack.push(1);
    stack.push(2);
    stack.push(3);
    
    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
}
```

## Real-World Examples

### State Machine

```rust
#[derive(Debug, Clone, PartialEq)]
enum ConnectionState {
    Disconnected,
    Connecting { attempt: u32 },
    Connected { session_id: String },
    Disconnecting { reason: String },
}

impl ConnectionState {
    fn connect(&mut self) -> Result<(), String> {
        match self {
            ConnectionState::Disconnected => {
                *self = ConnectionState::Connecting { attempt: 1 };
                Ok(())
            }
            ConnectionState::Connecting { attempt } => {
                if *attempt < 3 {
                    *self = ConnectionState::Connecting { attempt: attempt + 1 };
                    Ok(())
                } else {
                    Err("Max connection attempts reached".to_string())
                }
            }
            ConnectionState::Connected { .. } => {
                Err("Already connected".to_string())
            }
            ConnectionState::Disconnecting { .. } => {
                Err("Currently disconnecting".to_string())
            }
        }
    }
    
    fn disconnect(&mut self, reason: &str) {
        match self {
            ConnectionState::Connected { .. } => {
                *self = ConnectionState::Disconnecting {
                    reason: reason.to_string(),
                };
            }
            _ => {
                *self = ConnectionState::Disconnected;
            }
        }
    }
    
    fn complete_disconnect(&mut self) {
        if let ConnectionState::Disconnecting { .. } = self {
            *self = ConnectionState::Disconnected;
        }
    }
}
```

### Expression Parser

```rust
#[derive(Debug, PartialEq)]
enum Token {
    Number(f64),
    Plus,
    Minus,
    Multiply,
    Divide,
    LeftParen,
    RightParen,
    EOF,
}

#[derive(Debug)]
enum Expression {
    Number(f64),
    BinaryOp {
        left: Box<Expression>,
        operator: Token,
        right: Box<Expression>,
    },
    UnaryOp {
        operator: Token,
        operand: Box<Expression>,
    },
}

impl Expression {
    fn evaluate(&self) -> Result<f64, String> {
        match self {
            Expression::Number(value) => Ok(*value),
            Expression::BinaryOp { left, operator, right } => {
                let left_val = left.evaluate()?;
                let right_val = right.evaluate()?;
                
                match operator {
                    Token::Plus => Ok(left_val + right_val),
                    Token::Minus => Ok(left_val - right_val),
                    Token::Multiply => Ok(left_val * right_val),
                    Token::Divide => {
                        if right_val == 0.0 {
                            Err("Division by zero".to_string())
                        } else {
                            Ok(left_val / right_val)
                        }
                    }
                    _ => Err("Invalid operator".to_string()),
                }
            }
            Expression::UnaryOp { operator, operand } => {
                let operand_val = operand.evaluate()?;
                match operator {
                    Token::Minus => Ok(-operand_val),
                    _ => Err("Invalid unary operator".to_string()),
                }
            }
        }
    }
}
```

### API Response Handler

```rust
#[derive(Debug)]
enum ApiResponse<T> {
    Success { data: T, status: u16 },
    Error { message: String, status: u16 },
    NetworkError { reason: String },
    Timeout,
}

impl<T> ApiResponse<T> {
    fn handle_response(&self) -> Result<&T, String> {
        match self {
            ApiResponse::Success { data, status } => {
                if *status == 200 {
                    Ok(data)
                } else {
                    Err(format!("Unexpected status: {}", status))
                }
            }
            ApiResponse::Error { message, status } => {
                Err(format!("API Error {}: {}", status, message))
            }
            ApiResponse::NetworkError { reason } => {
                Err(format!("Network error: {}", reason))
            }
            ApiResponse::Timeout => {
                Err("Request timeout".to_string())
            }
        }
    }
    
    fn map<U, F>(self, f: F) -> ApiResponse<U>
    where
        F: FnOnce(T) -> U,
    {
        match self {
            ApiResponse::Success { data, status } => {
                ApiResponse::Success {
                    data: f(data),
                    status,
                }
            }
            ApiResponse::Error { message, status } => {
                ApiResponse::Error { message, status }
            }
            ApiResponse::NetworkError { reason } => {
                ApiResponse::NetworkError { reason }
            }
            ApiResponse::Timeout => ApiResponse::Timeout,
        }
    }
}
```

## Best Practices

### 1. Use Enums for State Machines

```rust
enum DownloadState {
    Idle,
    Downloading { progress: f32 },
    Completed { file_path: String },
    Failed { error: String },
}
```

### 2. Prefer Exhaustive Matching

```rust
// Good: exhaustive
match value {
    Some(x) => process(x),
    None => handle_none(),
}

// Avoid: using unwrap without handling errors
// let x = value.unwrap();
```

### 3. Use Pattern Guards for Complex Conditions

```rust
match user {
    User::Admin if user.has_permission("delete") => delete_user(),
    User::Admin => println!("Admin without delete permission"),
    User::Regular => println!("Regular user"),
}
```

### 4. Leverage if let for Single Cases

```rust
// Instead of:
match result {
    Ok(value) => process(value),
    Err(_) => {},
}

// Use:
if let Ok(value) = result {
    process(value);
}
```

## Performance Considerations

- **Zero-cost abstractions**: Enums and pattern matching have minimal runtime overhead
- **Tagged unions**: Rust uses efficient memory layouts for enums
- **Exhaustiveness checking**: Compile-time guarantees prevent runtime errors
- **Pattern optimization**: Compiler optimizes common pattern matching cases

## Common Pitfalls

### 1. Forgetting the Default Case

```rust
// This won't compile - not exhaustive
// match value {
//     Some(x) => println!("Got {}", x),
// }

// Correct:
match value {
    Some(x) => println!("Got {}", x),
    None => (),
}
```

### 2. Shadowing Variables

```rust
let x = Some(5);

match x {
    Some(x) => {  // This shadows the outer x
        println!("Inner x: {}", x);  // Refers to the matched value
    }
    None => (),
}

// Outer x is still available here
```

### 3. Moving vs Borrowing in Patterns

```rust
let value = Some(String::from("hello"));

match value {
    Some(s) => println!("{}", s),  // Moves the String
    None => (),
}

// value is no longer valid here

// To borrow instead:
let value = Some(String::from("hello"));

match &value {
    Some(s) => println!("{}", s),  // Borrows the String
    None => (),
}

// value is still valid here
```

## Summary

Enums and pattern matching are among Rust's most powerful features:

- **Enums** model complex data and state machines
- **Pattern matching** provides exhaustive, safe handling of variants
- **Option and Result** replace null pointers and exceptions
- **if let and while let** provide concise pattern matching
- **Zero-cost abstractions** ensure performance

Mastering enums and pattern matching is essential for writing idiomatic, safe Rust code.