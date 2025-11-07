# Rust Testing Framework - Comprehensive Guide

## Overview

Rust has a robust built-in testing framework that enables developers to write comprehensive tests with minimal boilerplate. The testing ecosystem is designed to be ergonomic, fast, and integrated directly into the language toolchain.

## Types of Tests

### 1. Unit Tests

Unit tests are written in the same file as the code they test, typically in a `tests` module marked with `#[cfg(test)]`.

```rust
// src/lib.rs
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }

    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    #[should_panic]
    fn another() {
        panic!("Make this test fail");
    }
}
```

### 2. Integration Tests

Integration tests are external to your library and use your code as any external code would, through its public API.

```rust
// tests/integration_test.rs
use my_crate;

#[test]
fn test_add() {
    assert_eq!(my_crate::add(2, 3), 5);
}

#[test]
fn test_complex_operation() {
    let result = my_crate::complex_function();
    assert!(result.is_ok());
}
```

### 3. Documentation Tests

Rust can execute code examples in your documentation, ensuring your examples stay up-to-date.

```rust
/// Adds two numbers together.
///
/// # Examples
///
/// ```
/// let result = my_crate::add(2, 2);
/// assert_eq!(result, 4);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

## Test Organization

### Directory Structure

```
my_project/
├── Cargo.toml
├── src/
│   ├── lib.rs          # Library crate
│   └── main.rs         # Binary crate
└── tests/
    ├── integration_test.rs
    ├── another_test.rs
    └── common/
        └── mod.rs      # Shared test utilities
```

### Common Test Module Pattern

```rust
// tests/common/mod.rs
pub fn setup() {
    // Setup code for multiple tests
    println!("Setting up test environment");
}

// tests/integration_test.rs
mod common;

#[test]
fn test_with_setup() {
    common::setup();
    // Test logic here
}
```

## Test Macros and Functions

### Assertion Macros

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn test_assertions() {
        // Basic boolean assertion
        assert!(true);
        
        // Equality assertions
        assert_eq!(2 + 2, 4);
        assert_ne!(3, 4);
        
        // Custom failure messages
        let result = 42;
        assert!(result > 40, "Result was {}, expected > 40", result);
        
        // Floating point comparisons (with epsilon)
        let x = 1.2331f64;
        let y = 1.2332f64;
        assert!((x - y).abs() < 1.0e-4);
    }
}
```

### Test Attributes

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn normal_test() {
        // This test runs normally
    }

    #[test]
    #[should_panic]
    fn should_panic_test() {
        panic!("This is expected!");
    }

    #[test]
    #[should_panic(expected = "assertion failed")]
    fn specific_panic_test() {
        assert!(false, "assertion failed");
    }

    #[test]
    #[ignore]
    fn expensive_test() {
        // This test is ignored by default
        // Run with: cargo test -- --ignored
    }
}
```

## Advanced Testing Patterns

### Test Fixtures

```rust
struct TestDatabase {
    connection: String,
}

impl TestDatabase {
    fn new() -> Self {
        Self {
            connection: "test_db".to_string(),
        }
    }
    
    fn cleanup(&self) {
        // Clean up test data
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_with_fixture() {
        let db = TestDatabase::new();
        
        // Use the test fixture
        assert_eq!(db.connection, "test_db");
        
        // Cleanup is automatic when db goes out of scope
    }
}
```

### Mocking and Dependency Injection

```rust
pub trait Database {
    fn get_user(&self, id: u32) -> Option<String>;
}

pub struct RealDatabase;

impl Database for RealDatabase {
    fn get_user(&self, id: u32) -> Option<String> {
        // Real database implementation
        Some(format!("User {}", id))
    }
}

pub struct MockDatabase;

impl Database for MockDatabase {
    fn get_user(&self, id: u32) -> Option<String> {
        // Mock implementation for testing
        if id == 1 {
            Some("Test User".to_string())
        } else {
            None
        }
    }
}

pub fn get_username(db: &dyn Database, id: u32) -> String {
    db.get_user(id).unwrap_or_else(|| "Unknown".to_string())
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_with_mock() {
        let mock_db = MockDatabase;
        let username = get_username(&mock_db, 1);
        assert_eq!(username, "Test User");
    }
}
```

## Property-Based Testing

Using libraries like `proptest` for property-based testing:

```rust
// Add to Cargo.toml: proptest = "1.0"

#[cfg(test)]
mod tests {
    use proptest::prelude::*;

    proptest! {
        #[test]
        fn addition_is_commutative(a in 0..1000i32, b in 0..1000i32) {
            assert_eq!(a + b, b + a);
        }

        #[test]
        fn string_concat_length(a in ".*", b in ".*") {
            let concat = format!("{}{}", a, b);
            assert_eq!(concat.len(), a.len() + b.len());
        }
    }
}
```

## Benchmark Tests

Rust supports built-in benchmarking (requires nightly Rust):

```rust
#![feature(test)]
extern crate test;

use test::Bencher;

#[bench]
fn bench_vector_creation(b: &mut Bencher) {
    b.iter(|| {
        let v: Vec<i32> = (0..1000).collect();
        v
    });
}

#[bench]
fn bench_string_concatenation(b: &mut Bencher) {
    let s1 = "hello".to_string();
    let s2 = "world".to_string();
    
    b.iter(|| {
        let s3 = s1.clone() + &s2;
        s3
    });
}
```

## Test Configuration and Running

### Running Specific Tests

```bash
# Run all tests
cargo test

# Run specific test by name
cargo test test_name

# Run tests containing a pattern
cargo test add

# Run ignored tests
cargo test -- --ignored

# Run tests in a single thread
cargo test -- --test-threads=1

# Show output of successful tests
cargo test -- --nocapture
```

### Test Configuration in Cargo.toml

```toml
[package]
name = "my_project"
version = "0.1.0"

[dev-dependencies]
proptest = "1.0"

# Test-specific configuration
[[test]]
name = "integration_test"
harness = false  # Disable default test harness

[profile.test]
debug = true  # Include debug info in test builds
```

## Best Practices

### 1. Test Organization
- Keep unit tests close to the code they test
- Use integration tests for public API testing
- Group related tests in modules

### 2. Test Naming
- Use descriptive test names
- Follow the pattern: `test_<function>_<scenario>`
- Make test failures easy to understand

### 3. Test Isolation
- Each test should be independent
- Avoid shared mutable state between tests
- Use setup/teardown for expensive operations

### 4. Test Coverage
- Test both success and failure cases
- Test edge cases and boundary conditions
- Use property-based testing for complex logic

### 5. Performance
- Keep tests fast
- Use mocks for external dependencies
- Avoid I/O operations in unit tests

## Common Testing Patterns

### Testing Error Cases

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_error_handling() {
        let result = fallible_function();
        
        match result {
            Ok(_) => panic!("Expected error"),
            Err(e) => {
                assert!(e.to_string().contains("expected error"));
            }
        }
    }
    
    #[test]
    fn test_result_assertions() {
        let result: Result<i32, &str> = Err("error message");
        
        assert!(result.is_err());
        assert_eq!(result.unwrap_err(), "error message");
    }
}
```

### Testing Async Code

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use tokio::runtime::Runtime;

    #[test]
    fn test_async_function() {
        let rt = Runtime::new().unwrap();
        
        rt.block_on(async {
            let result = async_function().await;
            assert_eq!(result, 42);
        });
    }
}
```

## Testing Tools and Libraries

### Popular Testing Crates

- **proptest**: Property-based testing
- **mockall**: Mocking library
- **criterion**: Benchmarking library
- **quickcheck**: Another property-based testing library
- **test-case**: Parameterized testing

### Example with Mockall

```toml
# Cargo.toml
[dev-dependencies]
mockall = "0.11"
```

```rust
#[cfg(test)]
mod tests {
    use mockall::*;
    use mockall::predicate::*;

    #[automock]
    trait MyTrait {
        fn do_something(&self, input: i32) -> i32;
    }

    #[test]
    fn test_with_mockall() {
        let mut mock = MockMyTrait::new();
        
        mock.expect_do_something()
            .with(eq(42))
            .times(1)
            .returning(|x| x * 2);
        
        let result = mock.do_something(42);
        assert_eq!(result, 84);
    }
}
```

## Conclusion

Rust's testing framework provides comprehensive tools for writing reliable, maintainable tests. The integration with Cargo makes running tests seamless, and the ecosystem offers powerful libraries for advanced testing scenarios. By following Rust's testing best practices, developers can build confidence in their code's correctness and maintainability.