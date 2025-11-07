# Rust Error Handling - Comprehensive Guide

## Introduction to Error Handling

Rust takes a unique approach to error handling that emphasizes safety and explicitness. Unlike languages with exceptions, Rust uses the type system to force developers to handle potential errors at compile time.

## Two Kinds of Errors

### Recoverable Errors

Use `Result<T, E>` for errors that can be reasonably handled:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### Unrecoverable Errors

Use `panic!` for errors that should stop program execution:

```rust
fn unrecoverable_error() {
    panic!("This is a catastrophic failure!");
}
```

## The Result Type

### Basic Usage

```rust
use std::fs::File;
use std::io::Error;

fn basic_result() -> Result<File, Error> {
    let file = File::open("hello.txt")?;
    Ok(file)
}

fn handle_result() {
    match basic_result() {
        Ok(file) => {
            println!("File opened successfully");
            // Use the file
        }
        Err(error) => {
            eprintln!("Failed to open file: {}", error);
        }
    }
}
```

### Common Result Methods

```rust
fn result_methods_examples() {
    let success: Result<i32, &str> = Ok(42);
    let failure: Result<i32, &str> = Err("Something went wrong");
    
    // Unwrapping with defaults
    let value1 = success.unwrap_or(0);        // 42
    let value2 = failure.unwrap_or(0);        // 0
    
    // Unwrapping with custom error message
    let value3 = success.expect("This should never fail"); // 42
    
    // Mapping values
    let doubled = success.map(|v| v * 2);     // Ok(84)
    let still_err = failure.map(|v| v * 2);   // Err("Something went wrong")
    
    // Mapping errors
    let mapped_err = failure.map_err(|e| format!("Error: {}", e));
    
    // And then chaining
    let result = success
        .and_then(|v| if v > 0 { Ok(v) } else { Err("Negative") })
        .map(|v| v * 2);
    
    // Checking if result is Ok or Err
    println!("Is success ok? {}", success.is_ok());     // true
    println!("Is failure ok? {}", failure.is_ok());     // false
    println!("Is success err? {}", success.is_err());   // false
    println!("Is failure err? {}", failure.is_err());   // true
}
```

## Error Propagation

### The ? Operator

The `?` operator is Rust's concise way to propagate errors:

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();
    
    // Without ? operator (verbose)
    let mut file = match File::open("username.txt") {
        Ok(file) => file,
        Err(e) => return Err(e),
    };
    
    // With ? operator (concise)
    file.read_to_string(&mut username)?;
    
    Ok(username)
}

// Even more concise with std::fs::read_to_string
fn read_username_concise() -> Result<String, io::Error> {
    std::fs::read_to_string("username.txt")
}
```

### Chaining with ?

```rust
use std::fs::File;
use std::io::{self, Write};

fn write_config() -> Result<(), io::Error> {
    let mut file = File::create("config.txt")?;
    file.write_all(b"config data")?;
    file.sync_all()?;
    Ok(())
}

fn complex_operation() -> Result<String, io::Error> {
    let content = std::fs::read_to_string("input.txt")?;
    let processed = process_content(&content)?;
    std::fs::write("output.txt", &processed)?;
    Ok(processed)
}

fn process_content(content: &str) -> Result<String, io::Error> {
    if content.is_empty() {
        return Err(io::Error::new(io::ErrorKind::InvalidData, "Empty content"));
    }
    Ok(content.to_uppercase())
}
```

## Custom Error Types

### Simple Custom Error

```rust
#[derive(Debug)]
struct MyError {
    message: String,
}

impl MyError {
    fn new(msg: &str) -> Self {
        Self {
            message: msg.to_string(),
        }
    }
}

impl std::fmt::Display for MyError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "MyError: {}", self.message)
    }
}

impl std::error::Error for MyError {}

fn function_that_fails() -> Result<(), MyError> {
    Err(MyError::new("Something went wrong"))
}
```

### Enum-Based Errors

```rust
#[derive(Debug)]
enum MathError {
    DivisionByZero,
    NegativeSquareRoot,
    Overflow,
}

impl std::fmt::Display for MathError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            MathError::DivisionByZero => write!(f, "Division by zero"),
            MathError::NegativeSquareRoot => write!(f, "Square root of negative number"),
            MathError::Overflow => write!(f, "Arithmetic overflow"),
        }
    }
}

impl std::error::Error for MathError {}

fn divide(a: f64, b: f64) -> Result<f64, MathError> {
    if b == 0.0 {
        Err(MathError::DivisionByZero)
    } else {
        Ok(a / b)
    }
}

fn square_root(x: f64) -> Result<f64, MathError> {
    if x < 0.0 {
        Err(MathError::NegativeSquareRoot)
    } else {
        Ok(x.sqrt())
    }
}
```

### Complex Error Types with Context

```rust
use std::fmt;

#[derive(Debug)]
struct ConfigError {
    kind: ConfigErrorKind,
    context: String,
    source: Option<Box<dyn std::error::Error + 'static>>,
}

#[derive(Debug)]
enum ConfigErrorKind {
    FileNotFound,
    ParseError,
    ValidationError,
}

impl ConfigError {
    fn file_notfound(path: &str) -> Self {
        Self {
            kind: ConfigErrorKind::FileNotFound,
            context: format!("File not found: {}", path),
            source: None,
        }
    }
    
    fn parse_error(context: &str, source: impl std::error::Error + 'static) -> Self {
        Self {
            kind: ConfigErrorKind::ParseError,
            context: context.to_string(),
            source: Some(Box::new(source)),
        }
    }
}

impl fmt::Display for ConfigError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Config Error: {:?} - {}", self.kind, self.context)
    }
}

impl std::error::Error for ConfigError {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        self.source.as_ref().map(|e| e.as_ref())
    }
}
```

## Error Conversion

### Using From Trait

```rust
use std::num::ParseIntError;

#[derive(Debug)]
struct CalculationError {
    message: String,
}

impl From<ParseIntError> for CalculationError {
    fn from(error: ParseIntError) -> Self {
        Self {
            message: format!("Parse error: {}", error),
        }
    }
}

fn parse_and_calculate(input: &str) -> Result<i32, CalculationError> {
    let number: i32 = input.parse()?;  // ? automatically converts ParseIntError to CalculationError
    Ok(number * 2)
}
```

### Multiple Error Types

```rust
use std::fs::File;
use std::io;

#[derive(Debug)]
enum ApplicationError {
    Io(io::Error),
    Parse(String),
    Validation(String),
}

impl From<io::Error> for ApplicationError {
    fn from(error: io::Error) -> Self {
        ApplicationError::Io(error)
    }
}

impl From<std::num::ParseIntError> for ApplicationError {
    fn from(error: std::num::ParseIntError) -> Self {
        ApplicationError::Parse(format!("Parse error: {}", error))
    }
}

fn load_config() -> Result<String, ApplicationError> {
    let content = std::fs::read_to_string("config.txt")?;
    Ok(content)
}

fn parse_config_value(value: &str) -> Result<i32, ApplicationError> {
    let number: i32 = value.parse()?;
    if number < 0 {
        return Err(ApplicationError::Validation("Value must be positive".to_string()));
    }
    Ok(number)
}
```

## The anyhow Crate

### Simplified Error Handling

For applications where you don't need to handle specific error types:

```rust
// Add to Cargo.toml: anyhow = "1.0"

use anyhow::{Context, Result};

fn read_config_file() -> Result<String> {
    let path = "config.toml";
    let content = std::fs::read_to_string(path)
        .with_context(|| format!("Failed to read config file: {}", path))?;
    
    Ok(content)
}

fn process_data() -> Result<()> {
    let config = read_config_file()?;
    // Process config...
    Ok(())
}

fn main() -> Result<()> {
    process_data()?;
    Ok(())
}
```

## The thiserror Crate

### Derive-Based Error Types

For libraries where you want to define specific error types:

```rust
// Add to Cargo.toml: thiserror = "1.0"

use thiserror::Error;

#[derive(Error, Debug)]
enum DataError {
    #[error("IO error: {source}")]
    Io {
        #[from]
        source: std::io::Error,
    },
    
    #[error("Parse error at line {line}: {message}")]
    Parse {
        line: usize,
        message: String,
    },
    
    #[error("Validation error: {field} - {reason}")]
    Validation {
        field: String,
        reason: String,
    },
}

fn load_data() -> Result<String, DataError> {
    let content = std::fs::read_to_string("data.txt")?;
    Ok(content)
}

fn parse_line(line: &str, line_num: usize) -> Result<(), DataError> {
    if line.is_empty() {
        return Err(DataError::Parse {
            line: line_num,
            message: "Empty line".to_string(),
        });
    }
    Ok(())
}
```

## Panic Handling

### When to Use panic!

```rust
fn validate_input(input: &str) {
    if input.is_empty() {
        panic!("Input cannot be empty");
    }
}

fn unrecoverable_state() {
    // Only panic for truly unrecoverable errors
    if cfg!(debug_assertions) {
        panic!("This should never happen in debug mode");
    }
}
```

### Custom Panic Handlers

```rust
use std::panic;

fn setup_panic_handler() {
    panic::set_hook(Box::new(|panic_info| {
        eprintln!("Custom panic handler:");
        if let Some(location) = panic_info.location() {
            eprintln!("Panic occurred in file '{}' at line {}",
                     location.file(),
                     location.line());
        }
        
        if let Some(s) = panic_info.payload().downcast_ref::<&str>() {
            eprintln!("Panic message: {}", s);
        }
    }));
}

fn main() {
    setup_panic_handler();
    
    // This panic will use our custom handler
    // panic!("Test panic");
}
```

### Catching Panics

```rust
fn catch_panics() {
    let result = panic::catch_unwind(|| {
        // Code that might panic
        if rand::random() {
            panic!("Random panic!");
        }
        "Success"
    });
    
    match result {
        Ok(value) => println!("No panic: {}", value),
        Err(_) => println!("Caught a panic!"),
    }
}
```

## Real-World Error Handling Patterns

### Database Operations

```rust
use thiserror::Error;

#[derive(Error, Debug)]
enum DatabaseError {
    #[error("Connection error: {source}")]
    Connection { source: std::io::Error },
    
    #[error("Query error: {message}")]
    Query { message: String },
    
    #[error("Transaction error: {message}")]
    Transaction { message: String },
}

struct Database;

impl Database {
    fn connect() -> Result<Self, DatabaseError> {
        // Simulate connection
        Ok(Database)
    }
    
    fn query(&self, sql: &str) -> Result<Vec<String>, DatabaseError> {
        if sql.is_empty() {
            return Err(DatabaseError::Query {
                message: "Empty SQL query".to_string(),
            });
        }
        
        // Simulate query execution
        Ok(vec!["result1".to_string(), "result2".to_string()])
    }
    
    fn execute_transaction<F>(&self, operations: F) -> Result<(), DatabaseError>
    where
        F: FnOnce() -> Result<(), DatabaseError>,
    {
        // Begin transaction
        println!("Beginning transaction");
        
        let result = operations();
        
        match result {
            Ok(()) => {
                println!("Committing transaction");
                Ok(())
            }
            Err(e) => {
                println!("Rolling back transaction");
                Err(DatabaseError::Transaction {
                    message: format!("Transaction failed: {}", e),
                })
            }
        }
    }
}
```

### Web API Error Handling

```rust
use std::collections::HashMap;
use thiserror::Error;

#[derive(Error, Debug)]
enum ApiError {
    #[error("Authentication failed: {reason}")]
    Auth { reason: String },
    
    #[error("Resource not found: {resource}")]
    NotFound { resource: String },
    
    #[error("Validation error: {field} - {message}")]
    Validation { field: String, message: String },
    
    #[error("Internal server error")]
    Internal,
}

impl ApiError {
    fn status_code(&self) -> u16 {
        match self {
            ApiError::Auth { .. } => 401,
            ApiError::NotFound { .. } => 404,
            ApiError::Validation { .. } => 400,
            ApiError::Internal => 500,
        }
    }
    
    fn to_response(&self) -> HashMap<&'static str, String> {
        let mut response = HashMap::new();
        response.insert("error", self.to_string());
        response.insert("status", self.status_code().to_string());
        response
    }
}

struct ApiHandler;

impl ApiHandler {
    fn handle_request(&self, path: &str) -> Result<String, ApiError> {
        match path {
            "/api/users" => self.get_users(),
            "/api/users/1" => self.get_user(1),
            _ => Err(ApiError::NotFound {
                resource: path.to_string(),
            }),
        }
    }
    
    fn get_users(&self) -> Result<String, ApiError> {
        // Simulate database call
        Ok(r#"["user1", "user2"]"#.to_string())
    }
    
    fn get_user(&self, id: u32) -> Result<String, ApiError> {
        if id == 0 {
            return Err(ApiError::Validation {
                field: "id".to_string(),
                message: "ID must be positive".to_string(),
            });
        }
        
        // Simulate user lookup
        if id == 1 {
            Ok(r#"{"id": 1, "name": "Alice"}"#.to_string())
        } else {
            Err(ApiError::NotFound {
                resource: format!("user with id {}", id),
            })
        }
    }
}
```

## Best Practices

### 1. Use Result for Recoverable Errors

```rust
// Good: Use Result
fn parse_config() -> Result<Config, ConfigError> {
    // ...
}

// Avoid: Using panic! for recoverable errors
// fn parse_config() -> Config {
//     // Don't panic here
// }
```

### 2. Provide Context with Errors

```rust
use anyhow::Context;

fn load_user_data(user_id: u32) -> anyhow::Result<String> {
    let path = format!("users/{}.json", user_id);
    std::fs::read_to_string(&path)
        .with_context(|| format!("Failed to load user data for user {}", user_id))
}
```

### 3. Use Specific Error Types for Libraries

```rust
// In library code, use specific error types
#[derive(Debug, thiserror::Error)]
pub enum MyLibraryError {
    #[error("Configuration error: {message}")]
    Config { message: String },
    // ...
}
```

### 4. Handle Errors at Appropriate Levels

```rust
// Low-level: Return specific errors
fn read_file_low_level(path: &str) -> Result<String, std::io::Error> {
    std::fs::read_to_string(path)
}

// High-level: Convert to application errors
fn load_config_high_level() -> Result<Config, ApplicationError> {
    let content = read_file_low_level("config.toml")?;
    // ...
}
```

## Performance Considerations

- **Zero-cost error handling**: The `?` operator has minimal runtime overhead
- **No stack unwinding**: Result-based error handling doesn't involve stack unwinding
- **Compile-time checks**: Error handling is verified at compile time
- **Minimal allocation**: Well-designed error types avoid unnecessary allocations

## Common Pitfalls

### 1. Overusing unwrap()

```rust
// Avoid: Using unwrap() in production code
// let file = File::open("config.txt").unwrap();

// Prefer: Proper error handling
let file = File::open("config.txt")?;
// or
let file = File::open("config.txt").expect("Config file must exist");
```

### 2. Ignoring Errors

```rust
// Avoid: Ignoring potential errors
// let _ = some_function_that_returns_result();

// Prefer: Handle or propagate errors
some_function_that_returns_result()?;
// or
if let Err(e) = some_function_that_returns_result() {
    eprintln!("Warning: {}", e);
}
```

### 3. Inconsistent Error Types

```rust
// Avoid: Mixing different error types
// fn inconsistent() -> Result<String, ???> {
//     std::fs::read_to_string("file1.txt")?;  // io::Error
//     "value".parse()?;                      // ParseIntError
// }

// Prefer: Unified error type
fn consistent() -> Result<String, MyError> {
    let content = std::fs::read_to_string("file1.txt")?;
    let number: i32 = content.parse()?;
    Ok(number.to_string())
}
```

## Summary

Rust's error handling system provides:

- **Compile-time safety** through the type system
- **Explicit error handling** with Result and Option
- **Concise error propagation** with the `?` operator
- **Flexible error types** with custom error definitions
- **Zero-cost abstractions** for performance
- **Rich ecosystem** with crates like anyhow and thiserror

Mastering Rust's error handling is essential for writing robust, maintainable applications.