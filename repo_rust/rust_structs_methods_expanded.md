# Rust Structs and Methods - Comprehensive Guide

## Introduction to Structs

Structs (structures) are custom data types that let you name and package together multiple related values. They are fundamental building blocks in Rust for creating complex data structures.

## Basic Struct Definition

### Simple Struct

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

fn create_user() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    
    println!("User: {} <{}>", user1.username, user1.email);
}
```

### Field Init Shorthand

When variable names match field names:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,      // shorthand for email: email
        username,   // shorthand for username: username
        active: true,
        sign_in_count: 1,
    }
}
```

### Struct Update Syntax

Create a new instance from an existing one:

```rust
fn update_user_example() {
    let user1 = User {
        email: String::from("user1@example.com"),
        username: String::from("user1"),
        active: true,
        sign_in_count: 1,
    };
    
    let user2 = User {
        email: String::from("user2@example.com"),
        username: String::from("user2"),
        ..user1  // Use the rest of user1's fields
    };
    
    println!("User2 active: {}", user2.active);  // true (from user1)
}
```

## Tuple Structs

Structs without named fields:

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn tuple_struct_examples() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
    
    println!("Black: ({}, {}, {})", black.0, black.1, black.2);
    println!("Origin: ({}, {}, {})", origin.0, origin.1, origin.2);
}
```

## Unit-Like Structs

Structs without any fields:

```rust
struct AlwaysEqual;

fn unit_struct_example() {
    let subject = AlwaysEqual;
    // Useful for marker types or implementing traits
}
```

## Method Implementation

### Basic Methods

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Associated function (like static method)
    fn new(width: u32, height: u32) -> Self {
        Self { width, height }
    }
    
    // Method that takes immutable reference to self
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    // Method that takes mutable reference to self
    fn scale(&mut self, factor: u32) {
        self.width *= factor;
        self.height *= factor;
    }
    
    // Method that takes ownership of self
    fn into_tuple(self) -> (u32, u32) {
        (self.width, self.height)
    }
    
    // Method with parameters
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn method_examples() {
    let mut rect = Rectangle::new(30, 50);
    
    println!("Area: {}", rect.area());
    
    rect.scale(2);
    println!("Scaled area: {}", rect.area());
    
    let other = Rectangle::new(10, 10);
    println!("Can hold other: {}", rect.can_hold(&other));
    
    let tuple = rect.into_tuple();
    println!("As tuple: {:?}", tuple);
    // rect is no longer valid here
}
```

### Multiple impl Blocks

You can split method implementations across multiple blocks:

```rust
impl Rectangle {
    fn width(&self) -> u32 {
        self.width
    }
}

impl Rectangle {
    fn height(&self) -> u32 {
        self.height
    }
}

// This is useful for conditional compilation
#[cfg(feature = "debug")]
impl Rectangle {
    fn debug_info(&self) -> String {
        format!("Rectangle {{ width: {}, height: {} }}", self.width, self.height)
    }
}
```

## Associated Functions

Functions that don't take `self` as a parameter:

```rust
impl Rectangle {
    // Constructor pattern
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
    
    // Factory method
    fn from_string(s: &str) -> Option<Self> {
        let parts: Vec<&str> = s.split('x').collect();
        if parts.len() == 2 {
            let width = parts[0].parse().ok()?;
            let height = parts[1].parse().ok()?;
            Some(Self { width, height })
        } else {
            None
        }
    }
}

fn associated_function_examples() {
    let square = Rectangle::square(10);
    let rect = Rectangle::from_string("30x50").unwrap();
    
    println!("Square area: {}", square.area());
    println!("Rect area: {}", rect.area());
}
```

## Advanced Struct Patterns

### Builder Pattern

```rust
struct UserBuilder {
    username: Option<String>,
    email: Option<String>,
    sign_in_count: u64,
    active: bool,
}

impl UserBuilder {
    fn new() -> Self {
        Self {
            username: None,
            email: None,
            sign_in_count: 0,
            active: true,
        }
    }
    
    fn username(mut self, username: &str) -> Self {
        self.username = Some(username.to_string());
        self
    }
    
    fn email(mut self, email: &str) -> Self {
        self.email = Some(email.to_string());
        self
    }
    
    fn sign_in_count(mut self, count: u64) -> Self {
        self.sign_in_count = count;
        self
    }
    
    fn inactive(mut self) -> Self {
        self.active = false;
        self
    }
    
    fn build(self) -> Result<User, String> {
        let username = self.username.ok_or("Username is required")?;
        let email = self.email.ok_or("Email is required")?;
        
        Ok(User {
            username,
            email,
            sign_in_count: self.sign_in_count,
            active: self.active,
        })
    }
}

fn builder_example() {
    let user = UserBuilder::new()
        .username("alice")
        .email("alice@example.com")
        .sign_in_count(5)
        .build()
        .unwrap();
    
    println!("Built user: {}", user.username);
}
```

### Newtype Pattern

Wrap a type to provide different behavior or implement traits:

```rust
struct Email(String);

impl Email {
    fn new(email: &str) -> Result<Self, String> {
        if email.contains('@') {
            Ok(Self(email.to_string()))
        } else {
            Err("Invalid email format".to_string())
        }
    }
    
    fn as_str(&self) -> &str {
        &self.0
    }
}

struct Username(String);

impl Username {
    fn new(username: &str) -> Result<Self, String> {
        if username.len() >= 3 && username.len() <= 20 {
            Ok(Self(username.to_string()))
        } else {
            Err("Username must be 3-20 characters".to_string())
        }
    }
    
    fn as_str(&self) -> &str {
        &self.0
    }
}

struct ValidatedUser {
    username: Username,
    email: Email,
    active: bool,
}
```

## Generic Structs

```rust
struct Pair<T, U> {
    first: T,
    second: U,
}

impl<T, U> Pair<T, U> {
    fn new(first: T, second: U) -> Self {
        Self { first, second }
    }
    
    fn first(&self) -> &T {
        &self.first
    }
    
    fn second(&self) -> &U {
        &self.second
    }
}

// Specialized implementation for when both types are the same
impl<T> Pair<T, T> {
    fn swap(self) -> Self {
        Self {
            first: self.second,
            second: self.first,
        }
    }
}

fn generic_struct_example() {
    let pair1 = Pair::new(5, "hello");
    let pair2 = Pair::new(10, 20);
    
    println!("Pair1: {} and {}", pair1.first(), pair1.second());
    
    let swapped = pair2.swap();
    println!("Swapped: {} and {}", swapped.first(), swapped.second());
}
```

## Lifetime Annotations in Structs

```rust
struct Excerpt<'a> {
    part: &'a str,
}

impl<'a> Excerpt<'a> {
    fn new(text: &'a str) -> Self {
        let first_sentence = text.split('.').next().unwrap_or("");
        Self { part: first_sentence }
    }
    
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}

fn lifetime_example() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let excerpt = Excerpt::new(&novel);
    
    println!("Excerpt: {}", excerpt.part);
    println!("Returned: {}", excerpt.announce_and_return_part("Listen!"));
}
```

## Deriving Common Traits

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
struct Person {
    name: String,
    age: u32,
}

impl Person {
    fn new(name: &str, age: u32) -> Self {
        Self {
            name: name.to_string(),
            age,
        }
    }
    
    fn birthday(&mut self) {
        self.age += 1;
    }
}

fn derived_traits_example() {
    let mut alice = Person::new("Alice", 30);
    let alice_clone = alice.clone();
    
    println!("Alice: {:?}", alice);
    println!("Clone: {:?}", alice_clone);
    println!("Equal: {}", alice == alice_clone);
    
    alice.birthday();
    println!("After birthday: {:?}", alice);
}
```

## Real-World Examples

### Database Model

```rust
#[derive(Debug, Clone)]
struct DatabaseConnection {
    host: String,
    port: u16,
    username: String,
    password: String,
    database: String,
    connected: bool,
}

impl DatabaseConnection {
    fn new(host: &str, port: u16, username: &str, password: &str, database: &str) -> Self {
        Self {
            host: host.to_string(),
            port,
            username: username.to_string(),
            password: password.to_string(),
            database: database.to_string(),
            connected: false,
        }
    }
    
    fn connect(&mut self) -> Result<(), String> {
        // Simulate connection logic
        if self.host.is_empty() {
            return Err("Host cannot be empty".to_string());
        }
        
        self.connected = true;
        Ok(())
    }
    
    fn disconnect(&mut self) {
        self.connected = false;
    }
    
    fn is_connected(&self) -> bool {
        self.connected
    }
    
    fn connection_string(&self) -> String {
        format!(
            "postgres://{}:{}@{}:{}/{}",
            self.username, self.password, self.host, self.port, self.database
        )
    }
}

impl Drop for DatabaseConnection {
    fn drop(&mut self) {
        if self.connected {
            println!("Closing database connection to {}", self.host);
            self.disconnect();
        }
    }
}
```

### Game Entity System

```rust
#[derive(Debug, Clone)]
struct Vector2 {
    x: f64,
    y: f64,
}

impl Vector2 {
    fn new(x: f64, y: f64) -> Self {
        Self { x, y }
    }
    
    fn zero() -> Self {
        Self { x: 0.0, y: 0.0 }
    }
    
    fn magnitude(&self) -> f64 {
        (self.x * self.x + self.y * self.y).sqrt()
    }
    
    fn normalize(&self) -> Option<Self> {
        let mag = self.magnitude();
        if mag > 0.0 {
            Some(Self {
                x: self.x / mag,
                y: self.y / mag,
            })
        } else {
            None
        }
    }
    
    fn add(&self, other: &Vector2) -> Self {
        Self {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
    
    fn scale(&self, scalar: f64) -> Self {
        Self {
            x: self.x * scalar,
            y: self.y * scalar,
        }
    }
}

#[derive(Debug)]
struct GameObject {
    position: Vector2,
    velocity: Vector2,
    mass: f64,
    name: String,
}

impl GameObject {
    fn new(name: &str, position: Vector2, mass: f64) -> Self {
        Self {
            position,
            velocity: Vector2::zero(),
            mass,
            name: name.to_string(),
        }
    }
    
    fn apply_force(&mut self, force: Vector2, delta_time: f64) {
        // F = ma, so a = F/m
        let acceleration = force.scale(1.0 / self.mass);
        self.velocity = self.velocity.add(&acceleration.scale(delta_time));
    }
    
    fn update(&mut self, delta_time: f64) {
        self.position = self.position.add(&self.velocity.scale(delta_time));
    }
    
    fn distance_to(&self, other: &GameObject) -> f64 {
        let dx = self.position.x - other.position.x;
        let dy = self.position.y - other.position.y;
        (dx * dx + dy * dy).sqrt()
    }
}
```

## Best Practices

### 1. Use Constructors

```rust
impl User {
    fn new(username: &str, email: &str) -> Self {
        Self {
            username: username.to_string(),
            email: email.to_string(),
            sign_in_count: 0,
            active: true,
        }
    }
}
```

### 2. Provide Getter Methods

```rust
impl User {
    fn username(&self) -> &str {
        &self.username
    }
    
    fn email(&self) -> &str {
        &self.email
    }
    
    fn is_active(&self) -> bool {
        self.active
    }
}
```

### 3. Use Builder Pattern for Complex Construction

```rust
impl User {
    fn builder() -> UserBuilder {
        UserBuilder::new()
    }
}
```

### 4. Implement Common Traits

```rust
#[derive(Debug, Clone, PartialEq)]
struct User {
    // ... fields
}

impl std::fmt::Display for User {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{} <{}>", self.username, self.email)
    }
}
```

## Common Pitfalls

### 1. Forgetting to Make Fields Public

```rust
mod my_module {
    pub struct User {
        pub username: String,  // Must be pub to access outside module
        email: String,         // Private by default
    }
    
    impl User {
        pub fn new(username: &str, email: &str) -> Self {
            Self {
                username: username.to_string(),
                email: email.to_string(),
            }
        }
        
        // Provide access to private fields via methods
        pub fn email(&self) -> &str {
            &self.email
        }
    }
}
```

### 2. Unnecessary Cloning

```rust
// Avoid unnecessary clones
impl User {
    // Good: returns reference
    fn username(&self) -> &str {
        &self.username
    }
    
    // Only clone when necessary
    fn clone_username(&self) -> String {
        self.username.clone()
    }
}
```

## Performance Considerations

- **Stack allocation**: Small structs are allocated on the stack
- **Field ordering**: Rust may reorder fields for optimal memory layout
- **Copy vs Move**: Small structs can implement `Copy` for better performance
- **Method dispatch**: Static dispatch for methods (no runtime cost)

## Summary

Structs and methods are fundamental to Rust programming:

- **Structs** organize related data
- **Methods** provide behavior for structs
- **Associated functions** act like static methods
- **Builder pattern** enables complex construction
- **Generic structs** provide type flexibility
- **Lifetime annotations** ensure memory safety

Mastering structs and methods is essential for writing idiomatic, efficient Rust code.