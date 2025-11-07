# Rust Closures - Comprehensive Guide

## Introduction to Closures

Closures are anonymous functions that can capture their environment. They are similar to lambda functions in other languages and are a powerful feature for functional programming patterns in Rust.

## Basic Closure Syntax

### Simple Closures

```rust
fn closure_basics() {
    // Closure with explicit type annotations
    let add_one = |x: i32| -> i32 { x + 1 };
    
    // Closure with type inference
    let add_one_short = |x| x + 1;
    
    // Calling closures
    println!("add_one(5) = {}", add_one(5));        // 6
    println!("add_one_short(5) = {}", add_one_short(5)); // 6
    
    // Closure with multiple parameters
    let add = |x, y| x + y;
    println!("add(3, 4) = {}", add(3, 4));          // 7
    
    // Closure with no parameters
    let always_five = || 5;
    println!("always_five() = {}", always_five());  // 5
}
```

### Closure Type Inference

```rust
fn closure_type_inference() {
    // The first call determines the parameter and return types
    let example_closure = |x| x;
    
    let s = example_closure(String::from("hello"));
    // Now the closure is bound to String -> String
    // This would cause an error:
    // let n = example_closure(5);  // Error: expected String, found integer
    
    println!("s = {}", s);
}
```

## Capturing the Environment

### Immutable Capture

```rust
fn immutable_capture() {
    let x = 4;
    
    // Closure captures x immutably
    let equal_to_x = |z| z == x;
    
    println!("Can still use x: {}", x);  // x is still available
    println!("4 equal to x: {}", equal_to_x(4));  // true
    
    let y = 4;
    let z = 5;
    println!("{} equal to x: {}", y, equal_to_x(y));  // true
    println!("{} equal to x: {}", z, equal_to_x(z));  // false
}
```

### Mutable Capture

```rust
fn mutable_capture() {
    let mut count = 0;
    
    // Closure captures count mutably
    let mut increment = || {
        count += 1;
        println!("Count is now: {}", count);
    };
    
    increment();  // Count is now: 1
    increment();  // Count is now: 2
    increment();  // Count is now: 3
    
    // This would cause an error:
    // println!("Count: {}", count);  // Error: cannot borrow count as immutable
    
    // After the closure is done, we can use count again
    println!("Final count: {}", count);  // 3
}
```

### Taking Ownership

```rust
fn ownership_capture() {
    let vec = vec![1, 2, 3];
    
    // Closure takes ownership of vec
    let owns_vec = move || {
        println!("Vector: {:?}", vec);
        // vec is moved into the closure
    };
    
    owns_vec();
    
    // This would cause an error:
    // println!("Vector: {:?}", vec);  // Error: value moved
}
```

## Function Traits

### Fn, FnMut, and FnOnce

```rust
fn function_traits() {
    let x = "hello".to_string();
    
    // Fn: Immutable borrow of captured variables
    let print_x = || {
        println!("x = {}", x);
    };
    print_x();
    println!("x is still available: {}", x);  // x is still valid
    
    let mut y = 5;
    
    // FnMut: Mutable borrow of captured variables
    let mut increment_y = || {
        y += 1;
        println!("y = {}", y);
    };
    increment_y();  // y = 6
    // println!("y = {}", y);  // Error: y is borrowed mutably
    
    let z = "world".to_string();
    
    // FnOnce: Takes ownership of captured variables
    let consume_z = move || {
        println!("z = {}", z);
        // z is consumed
    };
    consume_z();
    // println!("z = {}", z);  // Error: z was moved
}
```

### Trait Bounds with Closures

```rust
fn apply_closure<F>(f: F, value: i32) -> i32
where
    F: Fn(i32) -> i32,
{
    f(value)
}

fn apply_mut_closure<F>(mut f: F, value: i32) -> i32
where
    F: FnMut(i32) -> i32,
{
    f(value)
}

fn apply_once_closure<F>(f: F, value: i32) -> i32
where
    F: FnOnce(i32) -> i32,
{
    f(value)
}

fn trait_bounds_example() {
    let add_one = |x| x + 1;
    println!("apply_closure: {}", apply_closure(add_one, 5));  // 6
    
    let mut counter = 0;
    let mut count_and_add = |x| {
        counter += 1;
        x + counter
    };
    println!("apply_mut_closure: {}", apply_mut_closure(count_and_add, 5));  // 6
    
    let consume_string = |x| {
        let s = String::from("hello");
        println!("{}", s);
        x
    };
    println!("apply_once_closure: {}", apply_once_closure(consume_string, 5));  // 5
}
```

## Practical Closure Patterns

### Iterator Adapters

```rust
fn iterator_adapters() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Using closures with iterator methods
    let doubled: Vec<i32> = numbers.iter().map(|x| x * 2).collect();
    println!("Doubled: {:?}", doubled);  // [2, 4, 6, 8, 10]
    
    let evens: Vec<&i32> = numbers.iter().filter(|&&x| x % 2 == 0).collect();
    println!("Evens: {:?}", evens);  // [2, 4]
    
    let sum: i32 = numbers.iter().fold(0, |acc, &x| acc + x);
    println!("Sum: {}", sum);  // 15
    
    // Chaining multiple operations
    let result: Vec<i32> = numbers
        .iter()
        .filter(|&&x| x > 2)
        .map(|&x| x * 3)
        .collect();
    println!("Filtered and mapped: {:?}", result);  // [9, 12, 15]
}
```

### Custom Sorting

```rust
fn custom_sorting() {
    let mut words = vec![
        "banana", "apple", "cherry", "date"
    ];
    
    // Sort by length
    words.sort_by(|a, b| a.len().cmp(&b.len()));
    println!("Sorted by length: {:?}", words);
    
    // Sort by second character
    words.sort_by(|a, b| {
        a.chars().nth(1).cmp(&b.chars().nth(1))
    });
    println!("Sorted by second char: {:?}", words);
    
    // Custom complex sorting
    let mut numbers = vec![5, 2, 8, 1, 9, 3];
    numbers.sort_by(|a, b| {
        // Sort by distance from 5
        let dist_a = (a - 5).abs();
        let dist_b = (b - 5).abs();
        dist_a.cmp(&dist_b)
    });
    println!("Sorted by distance from 5: {:?}", numbers);
}
```

### Event Handlers

```rust
struct Button {
    on_click: Option<Box<dyn Fn()>>,
}

impl Button {
    fn new() -> Self {
        Self { on_click: None }
    }
    
    fn set_on_click<F>(&mut self, f: F)
    where
        F: Fn() + 'static,
    {
        self.on_click = Some(Box::new(f));
    }
    
    fn click(&self) {
        if let Some(ref handler) = self.on_click {
            handler();
        }
    }
}

fn event_handler_example() {
    let mut button = Button::new();
    
    // Capture environment in closure
    let message = "Button clicked!".to_string();
    let click_count = std::cell::RefCell::new(0);
    
    button.set_on_click(move || {
        let mut count = click_count.borrow_mut();
        *count += 1;
        println!("{} (click #{})", message, count);
    });
    
    button.click();  // Button clicked! (click #1)
    button.click();  // Button clicked! (click #2)
    button.click();  // Button clicked! (click #3)
}
```

## Advanced Closure Patterns

### Returning Closures

```rust
fn returns_closure() -> impl Fn(i32) -> i32 {
    |x| x + 1
}

fn returns_closure_with_capture() -> impl Fn(i32) -> i32 {
    let y = 10;
    move |x| x + y
}

fn factory_function(multiplier: i32) -> impl Fn(i32) -> i32 {
    move |x| x * multiplier
}

fn returning_closures_example() {
    let add_one = returns_closure();
    println!("add_one(5) = {}", add_one(5));  // 6
    
    let add_ten = returns_closure_with_capture();
    println!("add_ten(5) = {}", add_ten(5));  // 15
    
    let double = factory_function(2);
    let triple = factory_function(3);
    println!("double(5) = {}", double(5));    // 10
    println!("triple(5) = {}", triple(5));    // 15
}
```

### Closure Structs

```rust
struct ClosureStruct {
    data: i32,
}

impl ClosureStruct {
    fn new(data: i32) -> Self {
        Self { data }
    }
    
    fn call(&self) -> impl Fn(i32) -> i32 + '_ {
        // Return a closure that borrows self
        move |x| x + self.data
    }
    
    fn call_mut(&mut self) -> impl FnMut(i32) -> i32 + '_ {
        // Return a closure that mutably borrows self
        move |x| {
            self.data += 1;
            x + self.data
        }
    }
}

fn closure_struct_example() {
    let cs = ClosureStruct::new(10);
    let closure = cs.call();
    println!("closure(5) = {}", closure(5));  // 15
    
    let mut cs_mut = ClosureStruct::new(10);
    let mut closure_mut = cs_mut.call_mut();
    println!("closure_mut(5) = {}", closure_mut(5));  // 16
    println!("closure_mut(5) = {}", closure_mut(5));  // 17
}
```

### Memoization Pattern

```rust
use std::collections::HashMap;
use std::hash::Hash;

struct Memoizer<F, A, R>
where
    F: Fn(A) -> R,
    A: Eq + Hash + Clone,
    R: Clone,
{
    calculation: F,
    values: HashMap<A, R>,
}

impl<F, A, R> Memoizer<F, A, R>
where
    F: Fn(A) -> R,
    A: Eq + Hash + Clone,
    R: Clone,
{
    fn new(calculation: F) -> Self {
        Self {
            calculation,
            values: HashMap::new(),
        }
    }
    
    fn value(&mut self, arg: A) -> R {
        if let Some(value) = self.values.get(&arg) {
            value.clone()
        } else {
            let value = (self.calculation)(arg.clone());
            self.values.insert(arg, value.clone());
            value
        }
    }
}

fn memoization_example() {
    let expensive_closure = |x: i32| {
        println!("Calculating slowly...");
        std::thread::sleep(std::time::Duration::from_millis(100));
        x * x
    };
    
    let mut memoizer = Memoizer::new(expensive_closure);
    
    println!("First call (should calculate): {}", memoizer.value(5));
    println!("Second call (should use cache): {}", memoizer.value(5));
    println!("Different argument (should calculate): {}", memoizer.value(6));
    println!("Cached argument (should use cache): {}", memoizer.value(5));
}
```

## Real-World Examples

### Configuration System

```rust
struct Config {
    validators: Vec<Box<dyn Fn(&str) -> Result<(), String>>>,
}

impl Config {
    fn new() -> Self {
        Self {
            validators: Vec::new(),
        }
    }
    
    fn add_validator<F>(&mut self, validator: F)
    where
        F: Fn(&str) -> Result<(), String> + 'static,
    {
        self.validators.push(Box::new(validator));
    }
    
    fn validate(&self, value: &str) -> Result<(), String> {
        for validator in &self.validators {
            validator(value)?;
        }
        Ok(())
    }
}

fn config_system_example() {
    let mut config = Config::new();
    
    // Add various validators using closures
    config.add_validator(|value: &str| {
        if value.is_empty() {
            Err("Value cannot be empty".to_string())
        } else {
            Ok(())
        }
    });
    
    config.add_validator(|value: &str| {
        if value.len() < 3 {
            Err("Value must be at least 3 characters".to_string())
        } else {
            Ok(())
        }
    });
    
    config.add_validator(|value: &str| {
        if value.chars().any(|c| !c.is_alphanumeric()) {
            Err("Value must be alphanumeric".to_string())
        } else {
            Ok(())
        }
    });
    
    // Test validation
    println!("Validating 'hello': {:?}", config.validate("hello"));
    println!("Validating 'a': {:?}", config.validate("a"));
    println!("Validating 'test!': {:?}", config.validate("test!"));
}
```

### Pipeline Processing

```rust
struct Pipeline<T> {
    processors: Vec<Box<dyn Fn(T) -> T>>,
}

impl<T> Pipeline<T> {
    fn new() -> Self {
        Self {
            processors: Vec::new(),
        }
    }
    
    fn add_processor<F>(&mut self, processor: F)
    where
        F: Fn(T) -> T + 'static,
    {
        self.processors.push(Box::new(processor));
    }
    
    fn process(&self, mut data: T) -> T {
        for processor in &self.processors {
            data = processor(data);
        }
        data
    }
}

fn pipeline_example() {
    let mut text_pipeline = Pipeline::new();
    
    // Add text processing steps
    text_pipeline.add_processor(|s: String| s.to_uppercase());
    text_pipeline.add_processor(|s: String| s.replace(" ", "_"));
    text_pipeline.add_processor(|s: String| format!("processed_{}", s));
    
    let result = text_pipeline.process("hello world".to_string());
    println!("Pipeline result: {}", result);  // processed_HELLO_WORLD
    
    let mut number_pipeline = Pipeline::new();
    
    // Add number processing steps
    number_pipeline.add_processor(|x: i32| x * 2);
    number_pipeline.add_processor(|x: i32| x + 10);
    number_pipeline.add_processor(|x: i32| x / 2);
    
    let result = number_pipeline.process(5);
    println!("Number pipeline result: {}", result);  // 10
}
```

## Performance Considerations

- **Inline optimization**: Closures are often inlined by the compiler
- **Zero-cost abstractions**: No runtime overhead compared to function pointers
- **Capture size**: Large captures can affect performance
- **Trait objects**: Using `dyn Fn` has some runtime cost due to dynamic dispatch

## Best Practices

1. **Use type inference** when possible for cleaner code
2. **Prefer immutable captures** unless mutation is necessary
3. **Use `move` keyword** when you need to transfer ownership to the closure
4. **Consider performance** when capturing large data structures
5. **Use appropriate trait bounds** (`Fn`, `FnMut`, `FnOnce`) in function signatures
6. **Document complex closures** for better code readability

## Common Pitfalls

### Moving vs Borrowing

```rust
fn move_vs_borrow() {
    let data = vec![1, 2, 3];
    
    // This closure borrows data immutably
    let print_data = || {
        println!("Data: {:?}", data);
    };
    print_data();
    println!("Data is still available: {:?}", data);  // OK
    
    // This closure moves data
    let consume_data = move || {
        println!("Data: {:?}", data);
    };
    consume_data();
    // println!("Data: {:?}", data);  // Error: value moved
}
```

### Lifetime Issues

```rust
fn lifetime_issue() {
    let closure;
    {
        let x = 5;
        // closure = || println!("{}", x);  // Error: x doesn't live long enough
        closure = move || println!("{}", x);  // OK: x is moved into closure
    }
    closure();
}
```

## Summary

Closures in Rust provide:

- **Concise syntax** for anonymous functions
- **Environment capture** for accessing outer variables
- **Flexible ownership** through different function traits
- **Zero-cost abstractions** for performance
- **Functional programming patterns** with iterators and combinators

Mastering closures is essential for writing idiomatic Rust code, especially when working with iterators, event handlers, and functional programming patterns.