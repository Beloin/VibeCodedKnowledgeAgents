# Rust Modules and Visibility - Comprehensive Guide

## Introduction to Rust's Module System

Rust's module system helps you organize code into logical units and control visibility. It's designed to enforce encapsulation and provide clear boundaries between different parts of your codebase.

## Basic Module Structure

### Defining Modules

```rust
// In lib.rs or main.rs
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {
            println!("Adding to waitlist");
        }
        
        fn seat_at_table() {
            // This function is private to the hosting module
        }
    }
    
    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();
    
    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

### Module Files

You can split modules into separate files:

```rust
// src/lib.rs
mod front_of_house;  // Declares the module

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}

// src/front_of_house.rs
pub mod hosting;

// src/front_of_house/hosting.rs
pub fn add_to_waitlist() {
    println!("Adding to waitlist");
}

fn seat_at_table() {
    // Private function
}
```

## Visibility Modifiers

### Public vs Private

```rust
mod my_module {
    // Public: accessible from outside the module
    pub struct PublicStruct {
        pub public_field: i32,
        private_field: i32,  // Private by default
    }
    
    impl PublicStruct {
        pub fn new() -> Self {
            Self {
                public_field: 0,
                private_field: 0,
            }
        }
        
        // Public method
        pub fn get_private_field(&self) -> i32 {
            self.private_field
        }
        
        // Private method
        fn set_private_field(&mut self, value: i32) {
            self.private_field = value;
        }
    }
    
    // Private: only accessible within this module
    struct PrivateStruct {
        field: i32,
    }
    
    // Public function
    pub fn public_function() {
        println!("This is public");
        private_function();  // Can call private functions
    }
    
    // Private function
    fn private_function() {
        println!("This is private");
    }
}

fn main() {
    my_module::public_function();
    
    let public_struct = my_module::PublicStruct::new();
    println!("Public field: {}", public_struct.public_field);
    // println!("{}", public_struct.private_field);  // Error: private field
    
    println!("Private field via method: {}", public_struct.get_private_field());
}
```

### Advanced Visibility

```rust
mod advanced_visibility {
    // Public within current crate
    pub(crate) struct CratePublic {
        pub field: i32,
    }
    
    // Public within parent module
    pub(super) struct SuperPublic {
        field: i32,
    }
    
    // Public within specific module path
    pub(in crate::advanced_visibility) struct InModulePublic {
        field: i32,
    }
    
    mod inner {
        // Can access SuperPublic from parent
        fn use_super_public() {
            let _ = super::SuperPublic { field: 42 };
        }
    }
}
```

## Use Declarations

### Bringing Paths into Scope

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
        pub fn seat_at_table() {}
    }
}

// Bring hosting into scope
use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::seat_at_table();
}

// You can also bring specific items into scope
use crate::front_of_house::hosting::add_to_waitlist;

pub fn quick_seating() {
    add_to_waitlist();  // No need for hosting:: prefix
}
```

### Re-exporting with pub use

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

// Re-export hosting so external code can use it directly
pub use crate::front_of_house::hosting;

// Now external code can do:
// use my_crate::hosting::add_to_waitlist;
```

### Common Use Patterns

```rust
use std::collections::HashMap;
use std::fmt::Result;
use std::io::Result as IoResult;  // Renaming to avoid conflict

fn function1() -> Result {
    // Uses std::fmt::Result
    Ok(())
}

fn function2() -> IoResult<()> {
    // Uses std::io::Result
    Ok(())
}

// Nested paths
use std::{cmp::Ordering, io};
use std::io::{self, Write};  // Brings std::io and std::io::Write

// Glob operator (use with caution)
use std::collections::*;
```

## File Organization Patterns

### Library Structure

```
my_library/
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── config.rs
    ├── database/
    │   ├── mod.rs
    │   ├── connection.rs
    │   └── query.rs
    ├── models/
    │   ├── mod.rs
    │   ├── user.rs
    │   └── product.rs
    └── utils/
        ├── mod.rs
        └── validation.rs
```

### Example Implementation

```rust
// src/lib.rs
pub mod config;
pub mod database;
pub mod models;
pub mod utils;

// Re-export commonly used items
pub use database::Database;
pub use models::{User, Product};

// src/config.rs
#[derive(Debug)]
pub struct Config {
    pub database_url: String,
    pub port: u16,
}

impl Config {
    pub fn new() -> Result<Self, String> {
        Ok(Self {
            database_url: std::env::var("DATABASE_URL")
                .unwrap_or_else(|_| "postgres://localhost:5432".to_string()),
            port: std::env::var("PORT")
                .ok()
                .and_then(|p| p.parse().ok())
                .unwrap_or(8080),
        })
    }
}

// src/database/mod.rs
pub mod connection;
pub mod query;

pub use connection::DatabaseConnection;
pub use query::QueryBuilder;

// src/database/connection.rs
use crate::config::Config;

pub struct DatabaseConnection {
    url: String,
    connected: bool,
}

impl DatabaseConnection {
    pub fn new(config: &Config) -> Self {
        Self {
            url: config.database_url.clone(),
            connected: false,
        }
    }
    
    pub fn connect(&mut self) -> Result<(), String> {
        // Connection logic
        self.connected = true;
        Ok(())
    }
    
    pub fn is_connected(&self) -> bool {
        self.connected
    }
}

// src/models/mod.rs
pub mod user;
pub mod product;

pub use user::User;
pub use product::Product;

// src/models/user.rs
#[derive(Debug, Clone)]
pub struct User {
    pub id: u64,
    pub username: String,
    pub email: String,
}

impl User {
    pub fn new(id: u64, username: &str, email: &str) -> Self {
        Self {
            id,
            username: username.to_string(),
            email: email.to_string(),
        }
    }
}
```

## Advanced Module Patterns

### Conditional Compilation

```rust
#[cfg(feature = "database")]
pub mod database {
    pub fn connect() -> String {
        "Connected to database".to_string()
    }
}

#[cfg(not(feature = "database"))]
pub mod database {
    pub fn connect() -> String {
        "Database feature disabled".to_string()
    }
}

// Platform-specific modules
#[cfg(target_os = "windows")]
mod windows_impl {
    pub fn get_system_info() -> String {
        "Windows system info".to_string()
    }
}

#[cfg(target_os = "linux")]
mod linux_impl {
    pub fn get_system_info() -> String {
        "Linux system info".to_string()
    }
}
```

### Private Module Pattern

```rust
// Public API
pub struct PublicApi {
    inner: PrivateImplementation,
}

impl PublicApi {
    pub fn new() -> Self {
        Self {
            inner: PrivateImplementation::new(),
        }
    }
    
    pub fn public_method(&self) -> String {
        self.inner.private_method()
    }
}

// Private implementation details
struct PrivateImplementation {
    data: String,
}

impl PrivateImplementation {
    fn new() -> Self {
        Self {
            data: "private data".to_string(),
        }
    }
    
    fn private_method(&self) -> String {
        self.data.clone()
    }
}
```

### Module-Level Documentation

```rust
//! # My Library
//! 
//! This is a comprehensive library for doing amazing things.
//! 
//! ## Examples
//! 
//! ```rust
//! use my_library::PublicApi;
//! 
//! let api = PublicApi::new();
//! println!("{}", api.public_method());
//! ```

/// Configuration module for the library.
/// 
/// This module contains all configuration-related types and functions.
pub mod config {
    /// The main configuration struct.
    /// 
    /// # Examples
    /// 
    /// ```
    /// use my_library::config::Config;
    /// 
    /// let config = Config::new();
    /// ```
    #[derive(Debug)]
    pub struct Config {
        /// Database connection URL
        pub database_url: String,
    }
    
    impl Config {
        /// Creates a new configuration with default values.
        pub fn new() -> Self {
            Self {
                database_url: "postgres://localhost:5432".to_string(),
            }
        }
    }
}
```

## Workspace Organization

### Multi-Crate Projects

```toml
# Cargo.toml (workspace root)
[workspace]
members = [
    "crates/core",
    "crates/cli",
    "crates/web",
]

# crates/core/Cargo.toml
[package]
name = "my-core"
version = "0.1.0"

[dependencies]
# Other dependencies...

# crates/cli/Cargo.toml
[package]
name = "my-cli"
version = "0.1.0"

[dependencies]
my-core = { path = "../core" }
```

### Internal Dependencies

```rust
// In crates/cli/src/main.rs
use my_core::config::Config;
use my_core::database::DatabaseConnection;

fn main() {
    let config = Config::new();
    let mut db = DatabaseConnection::new(&config);
    
    if let Err(e) = db.connect() {
        eprintln!("Failed to connect: {}", e);
        return;
    }
    
    println!("Connected successfully!");
}
```

## Testing Organization

### Unit Tests

```rust
// In the same file as the code being tested
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_add() {
        assert_eq!(add(2, 2), 4);
    }
    
    #[test]
    #[should_panic]
    fn test_add_overflow() {
        add(i32::MAX, 1);
    }
}
```

### Integration Tests

```rust
// tests/integration_test.rs
use my_library;

#[test]
fn test_public_api() {
    let api = my_library::PublicApi::new();
    assert!(!api.public_method().is_empty());
}

// tests/common/mod.rs
pub fn setup_test_database() -> String {
    "test_database_url".to_string()
}

// tests/database_test.rs
mod common;

#[test]
fn test_database_operations() {
    let db_url = common::setup_test_database();
    // Test database operations
}
```

## Best Practices

### 1. Use Clear Module Hierarchy

```rust
// Good: Clear hierarchy
mod network {
    pub mod http {
        pub mod client;
        pub mod server;
    }
    pub mod tcp;
}

// Avoid: Flat structure with unclear relationships
// mod http_client;
// mod http_server;
// mod tcp_client;
// mod tcp_server;
```

### 2. Control Visibility Carefully

```rust
mod api {
    // Public interface
    pub struct PublicApi {
        inner: PrivateImpl,
    }
    
    impl PublicApi {
        pub fn new() -> Self { /* ... */ }
        pub fn public_method(&self) { /* ... */ }
    }
    
    // Private implementation
    struct PrivateImpl {
        // Implementation details
    }
    
    impl PrivateImpl {
        fn private_method(&self) { /* ... */ }
    }
}
```

### 3. Use Re-exports for Clean APIs

```rust
// src/lib.rs
mod internal {
    pub mod complex {
        pub mod implementation {
            pub struct ComplexType { /* ... */ }
            pub fn complex_function() { /* ... */ }
        }
    }
}

// Re-export for clean public API
pub use internal::complex::implementation::ComplexType;
pub use internal::complex::implementation::complex_function as simple_function;
```

### 4. Document Public APIs

```rust
/// A user of the system.
/// 
/// # Examples
/// 
/// ```
/// use my_library::User;
/// 
/// let user = User::new(1, "alice");
/// ```
#[derive(Debug)]
pub struct User {
    /// The unique identifier for the user.
    pub id: u64,
    /// The user's display name.
    pub username: String,
}

impl User {
    /// Creates a new user with the given ID and username.
    pub fn new(id: u64, username: &str) -> Self {
        Self {
            id,
            username: username.to_string(),
        }
    }
}
```

## Common Pitfalls

### 1. Circular Dependencies

```rust
// Avoid: Circular module dependencies
// mod a { use super::b; }
// mod b { use super::a; }

// Solution: Restructure or use traits
mod common {
    pub trait CommonTrait { /* ... */ }
}

mod a {
    use super::common::CommonTrait;
    // Implement CommonTrait
}

mod b {
    use super::common::CommonTrait;
    // Use CommonTrait
}
```

### 2. Overly Deep Module Hierarchies

```rust
// Avoid: Too many levels
// mod a { mod b { mod c { mod d { /* ... */ } } } }

// Prefer: Flatter structure when possible
mod a_b_c_d {
    // Combine related functionality
}
```

### 3. Inconsistent Naming

```rust
// Good: Consistent naming
mod user_management;
mod product_catalog;
mod order_processing;

// Avoid: Inconsistent naming
// mod users;
// mod product_management;
// mod process_orders;
```

## Performance Considerations

- **Compile times**: Deep module hierarchies can increase compile times
- **Binary size**: Unused modules are optimized away by the compiler
- **Runtime performance**: Module organization has no runtime cost
- **Incremental compilation**: Proper module boundaries help with incremental builds

## Summary

Rust's module system provides:

- **Clear code organization** through modules and files
- **Fine-grained visibility control** with pub modifiers
- **Clean public APIs** with use and pub use
- **Encapsulation** through private implementation details
- **Workspace support** for multi-crate projects
- **Testing organization** with built-in test modules

Mastering Rust's module system is essential for building maintainable, well-organized codebases.