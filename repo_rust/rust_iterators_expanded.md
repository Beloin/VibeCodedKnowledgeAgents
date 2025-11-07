# Rust Iterators - Comprehensive Guide

## Introduction to Iterators

Iterators are a fundamental abstraction in Rust that allow you to process sequences of items. They provide a clean, functional way to work with collections and are zero-cost abstractions that compile to efficient code.

## Basic Iterator Usage

### Creating Iterators

```rust
fn basic_iterator_creation() {
    let v1 = vec![1, 2, 3];
    
    // Different ways to create iterators
    let iter1 = v1.iter();        // Immutable references
    let iter2 = v1.iter_mut();    // Mutable references
    let iter3 = v1.into_iter();   // Takes ownership
    
    // From ranges
    let range_iter = (1..10).into_iter();
    
    // From arrays
    let array = [1, 2, 3, 4, 5];
    let array_iter = array.iter();
    
    // From strings
    let s = "hello";
    let char_iter = s.chars();
    let byte_iter = s.bytes();
}
```

### Consuming Iterators

```rust
fn consuming_iterators() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // for loop (automatically calls into_iter)
    for num in &numbers {
        println!("Number: {}", num);
    }
    
    // Manual iteration with next()
    let mut iter = numbers.iter();
    while let Some(num) = iter.next() {
        println!("Manual: {}", num);
    }
    
    // Consuming methods
    let sum: i32 = numbers.iter().sum();
    println!("Sum: {}", sum);
    
    let count = numbers.iter().count();
    println!("Count: {}", count);
    
    let last = numbers.iter().last();
    println!("Last: {:?}", last);
}
```

## Iterator Adapters

### Transforming Data

```rust
fn transforming_iterators() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Map - transform each element
    let doubled: Vec<i32> = numbers.iter().map(|x| x * 2).collect();
    println!("Doubled: {:?}", doubled);  // [2, 4, 6, 8, 10]
    
    // Filter - keep only elements that match a condition
    let evens: Vec<&i32> = numbers.iter().filter(|&&x| x % 2 == 0).collect();
    println!("Evens: {:?}", evens);  // [2, 4]
    
    // Filter_map - filter and transform in one operation
    let parsed: Vec<i32> = vec!["1", "2", "three", "4"]
        .iter()
        .filter_map(|s| s.parse().ok())
        .collect();
    println!("Parsed: {:?}", parsed);  // [1, 2, 4]
    
    // Flat_map - transform each element to iterator and flatten
    let matrix = vec![vec![1, 2], vec![3, 4], vec![5, 6]];
    let flattened: Vec<i32> = matrix.iter().flat_map(|row| row.iter().cloned()).collect();
    println!("Flattened: {:?}", flattened);  // [1, 2, 3, 4, 5, 6]
}
```

### Combining Iterators

```rust
fn combining_iterators() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Chain - combine multiple iterators
    let more_numbers = vec![6, 7, 8];
    let chained: Vec<i32> = numbers.iter().chain(more_numbers.iter()).cloned().collect();
    println!("Chained: {:?}", chained);  // [1, 2, 3, 4, 5, 6, 7, 8]
    
    // Zip - combine two iterators into pairs
    let names = vec!["Alice", "Bob", "Charlie"];
    let ages = vec![25, 30, 35];
    let zipped: Vec<(&str, i32)> = names.iter().zip(ages.iter()).map(|(n, &a)| (*n, a)).collect();
    println!("Zipped: {:?}", zipped);  // [("Alice", 25), ("Bob", 30), ("Charlie", 35)]
    
    // Enumerate - get (index, value) pairs
    let enumerated: Vec<(usize, &i32)> = numbers.iter().enumerate().collect();
    println!("Enumerated: {:?}", enumerated);  // [(0, 1), (1, 2), (2, 3), (3, 4), (4, 5)]
}
```

### Finding and Searching

```rust
fn finding_elements() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Find first element matching condition
    let first_even = numbers.iter().find(|&&x| x % 2 == 0);
    println!("First even: {:?}", first_even);  // Some(2)
    
    // Find position of element
    let position = numbers.iter().position(|&x| x == 3);
    println!("Position of 3: {:?}", position);  // Some(2)
    
    // Check if any element matches condition
    let has_negative = numbers.iter().any(|&x| x < 0);
    println!("Has negative: {}", has_negative);  // false
    
    // Check if all elements match condition
    let all_positive = numbers.iter().all(|&x| x > 0);
    println!("All positive: {}", all_positive);  // true
    
    // Find maximum and minimum
    let max = numbers.iter().max();
    let min = numbers.iter().min();
    println!("Max: {:?}, Min: {:?}", max, min);  // Some(5), Some(1)
}
```

## Advanced Iterator Patterns

### Folding and Reducing

```rust
fn folding_examples() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Fold - accumulate values with an initial value
    let sum = numbers.iter().fold(0, |acc, &x| acc + x);
    println!("Sum: {}", sum);  // 15
    
    let product = numbers.iter().fold(1, |acc, &x| acc * x);
    println!("Product: {}", product);  // 120
    
    // Reduce - like fold but uses first element as initial value
    let max = numbers.iter().cloned().reduce(|acc, x| if x > acc { x } else { acc });
    println!("Max: {:?}", max);  // Some(5)
    
    // Complex fold - build a string
    let joined = numbers.iter().fold(String::new(), |mut acc, &x| {
        if !acc.is_empty() {
            acc.push_str(", ");
        }
        acc.push_str(&x.to_string());
        acc
    });
    println!("Joined: {}", joined);  // "1, 2, 3, 4, 5"
}
```

### Grouping and Partitioning

```rust
fn grouping_examples() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // Partition - split into two collections based on predicate
    let (evens, odds): (Vec<i32>, Vec<i32>) = numbers
        .into_iter()
        .partition(|&x| x % 2 == 0);
    println!("Evens: {:?}, Odds: {:?}", evens, odds);
    
    // Take and skip
    let first_three: Vec<i32> = (1..=10).take(3).collect();
    println!("First three: {:?}", first_three);  // [1, 2, 3]
    
    let after_five: Vec<i32> = (1..=10).skip(5).collect();
    println!("After five: {:?}", after_five);  // [6, 7, 8, 9, 10]
    
    // Take while and skip while
    let until_five: Vec<i32> = (1..=10).take_while(|&x| x <= 5).collect();
    println!("Until five: {:?}", until_five);  // [1, 2, 3, 4, 5]
    
    let after_five_condition: Vec<i32> = (1..=10).skip_while(|&x| x <= 5).collect();
    println!("After five condition: {:?}", after_five_condition);  // [6, 7, 8, 9, 10]
}
```

### Lazy Evaluation

```rust
fn lazy_evaluation() {
    // Iterators are lazy - they don't do anything until consumed
    let numbers = 1..=1000000;
    
    // This doesn't actually process all million numbers
    let result = numbers
        .filter(|&x| x % 2 == 0)     // Only even numbers
        .map(|x| x * 2)              // Double them
        .take(5)                     // Only take first 5
        .collect::<Vec<i32>>();      // Now it executes
    
    println!("Result: {:?}", result);  // [4, 8, 12, 16, 20]
    
    // Infinite iterators
    let natural_numbers = 1..;
    let first_ten: Vec<i32> = natural_numbers.take(10).collect();
    println!("First ten natural numbers: {:?}", first_ten);
    
    // Cycle - repeat iterator infinitely
    let repeated = vec![1, 2, 3].into_iter().cycle().take(10).collect::<Vec<i32>>();
    println!("Repeated: {:?}", repeated);  // [1, 2, 3, 1, 2, 3, 1, 2, 3, 1]
}
```

## Creating Custom Iterators

### Implementing Iterator Trait

```rust
struct Counter {
    count: u32,
    max: u32,
}

impl Counter {
    fn new(max: u32) -> Self {
        Self { count: 0, max }
    }
}

impl Iterator for Counter {
    type Item = u32;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.count < self.max {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}

fn custom_iterator_example() {
    let counter = Counter::new(5);
    
    for num in counter {
        println!("Counter: {}", num);  // 1, 2, 3, 4, 5
    }
    
    // Can use all iterator methods
    let sum: u32 = Counter::new(10).sum();
    println!("Sum of first 10: {}", sum);  // 55
}
```

### Fibonacci Iterator

```rust
struct Fibonacci {
    current: u64,
    next: u64,
}

impl Fibonacci {
    fn new() -> Self {
        Self { current: 0, next: 1 }
    }
}

impl Iterator for Fibonacci {
    type Item = u64;
    
    fn next(&mut self) -> Option<Self::Item> {
        let new_next = self.current.checked_add(self.next)?;
        
        let result = self.current;
        self.current = self.next;
        self.next = new_next;
        
        Some(result)
    }
}

fn fibonacci_example() {
    let fib = Fibonacci::new();
    
    // First 10 Fibonacci numbers
    let first_ten: Vec<u64> = fib.take(10).collect();
    println!("First 10 Fibonacci: {:?}", first_ten);
    
    // Fibonacci numbers under 100
    let under_100: Vec<u64> = Fibonacci::new().take_while(|&x| x < 100).collect();
    println!("Fibonacci under 100: {:?}", under_100);
}
```

### Tree Iterator

```rust
use std::collections::VecDeque;

struct TreeNode<T> {
    value: T,
    children: Vec<TreeNode<T>>,
}

impl<T> TreeNode<T> {
    fn new(value: T) -> Self {
        Self {
            value,
            children: Vec::new(),
        }
    }
    
    fn add_child(&mut self, child: TreeNode<T>) {
        self.children.push(child);
    }
    
    // Depth-first traversal
    fn dfs_iter(&self) -> DepthFirstIter<T> {
        DepthFirstIter::new(self)
    }
    
    // Breadth-first traversal
    fn bfs_iter(&self) -> BreadthFirstIter<T> {
        BreadthFirstIter::new(self)
    }
}

struct DepthFirstIter<'a, T> {
    stack: Vec<&'a TreeNode<T>>,
}

impl<'a, T> DepthFirstIter<'a, T> {
    fn new(root: &'a TreeNode<T>) -> Self {
        Self { stack: vec![root] }
    }
}

impl<'a, T> Iterator for DepthFirstIter<'a, T> {
    type Item = &'a T;
    
    fn next(&mut self) -> Option<Self::Item> {
        let node = self.stack.pop()?;
        
        // Push children in reverse order for pre-order traversal
        for child in node.children.iter().rev() {
            self.stack.push(child);
        }
        
        Some(&node.value)
    }
}

struct BreadthFirstIter<'a, T> {
    queue: VecDeque<&'a TreeNode<T>>,
}

impl<'a, T> BreadthFirstIter<'a, T> {
    fn new(root: &'a TreeNode<T>) -> Self {
        let mut queue = VecDeque::new();
        queue.push_back(root);
        Self { queue }
    }
}

impl<'a, T> Iterator for BreadthFirstIter<'a, T> {
    type Item = &'a T;
    
    fn next(&mut self) -> Option<Self::Item> {
        let node = self.queue.pop_front()?;
        
        // Add children to the end of the queue
        for child in &node.children {
            self.queue.push_back(child);
        }
        
        Some(&node.value)
    }
}

fn tree_iterator_example() {
    let mut root = TreeNode::new(1);
    let mut child1 = TreeNode::new(2);
    let mut child2 = TreeNode::new(3);
    
    child1.add_child(TreeNode::new(4));
    child1.add_child(TreeNode::new(5));
    child2.add_child(TreeNode::new(6));
    
    root.add_child(child1);
    root.add_child(child2);
    
    println!("DFS: {:?}", root.dfs_iter().collect::<Vec<&i32>>());
    println!("BFS: {:?}", root.bfs_iter().collect::<Vec<&i32>>());
}
```

## Real-World Examples

### Data Processing Pipeline

```rust
struct DataPoint {
    timestamp: u64,
    value: f64,
    quality: u8,
}

impl DataPoint {
    fn new(timestamp: u64, value: f64, quality: u8) -> Self {
        Self {
            timestamp,
            value,
            quality,
        }
    }
}

fn data_processing_pipeline() {
    let data = vec![
        DataPoint::new(1000, 25.5, 1),
        DataPoint::new(1001, 26.0, 1),
        DataPoint::new(1002, 1000.0, 0),  // Bad quality
        DataPoint::new(1003, 27.5, 1),
        DataPoint::new(1004, 28.0, 1),
        DataPoint::new(1005, 500.0, 0),   // Bad quality
        DataPoint::new(1006, 29.5, 1),
    ];
    
    // Process data: filter bad quality, calculate statistics
    let processed_data: Vec<f64> = data
        .iter()
        .filter(|dp| dp.quality == 1)           // Only good quality data
        .map(|dp| dp.value)                     // Extract values
        .filter(|&v| v >= 0.0 && v <= 100.0)   // Remove outliers
        .collect();
    
    println!("Processed data: {:?}", processed_data);
    
    // Calculate statistics
    let count = processed_data.len();
    let sum: f64 = processed_data.iter().sum();
    let avg = sum / count as f64;
    let max = processed_data.iter().fold(f64::MIN, |a, &b| a.max(b));
    let min = processed_data.iter().fold(f64::MAX, |a, &b| a.min(b));
    
    println!("Count: {}, Avg: {:.2}, Min: {:.2}, Max: {:.2}", count, avg, min, max);
}
```

### Text Analysis

```rust
use std::collections::HashMap;

fn text_analysis(text: &str) {
    let word_counts: HashMap<&str, usize> = text
        .to_lowercase()
        .split_whitespace()
        .map(|word| word.trim_matches(|c: char| !c.is_alphabetic()))
        .filter(|word| !word.is_empty())
        .fold(HashMap::new(), |mut acc, word| {
            *acc.entry(word).or_insert(0) += 1;
            acc
        });
    
    // Find most common words
    let mut sorted_words: Vec<(&str, usize)> = word_counts.into_iter().collect();
    sorted_words.sort_by(|a, b| b.1.cmp(&a.1));
    
    println!("Top 5 words:");
    for (word, count) in sorted_words.iter().take(5) {
        println!("  {}: {}", word, count);
    }
    
    // Character statistics
    let char_count = text.chars().count();
    let letter_count = text.chars().filter(|c| c.is_alphabetic()).count();
    let digit_count = text.chars().filter(|c| c.is_numeric()).count();
    let space_count = text.chars().filter(|c| c.is_whitespace()).count();
    
    println!("\nCharacter statistics:");
    println!("  Total characters: {}", char_count);
    println!("  Letters: {}", letter_count);
    println!("  Digits: {}", digit_count);
    println!("  Spaces: {}", space_count);
}

fn text_analysis_example() {
    let text = "Hello world! This is a test. Hello again, world!";
    text_analysis(text);
}
```

## Performance Considerations

### Lazy Evaluation Benefits

```rust
fn lazy_evaluation_benefits() {
    let numbers = 1..=1_000_000;
    
    // This only processes until it finds the first even number > 100
    let result = numbers
        .filter(|&x| x % 2 == 0)
        .find(|&x| x > 100);
    
    println!("First even > 100: {:?}", result);  // Some(102)
    // Only processes 102 numbers, not 1 million!
}
```

### Avoiding Intermediate Collections

```rust
fn avoid_intermediate_collections() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // Inefficient: creates intermediate Vecs
    // let doubled: Vec<i32> = numbers.iter().map(|x| x * 2).collect();
    // let filtered: Vec<i32> = doubled.into_iter().filter(|x| x % 3 == 0).collect();
    
    // Efficient: chains operations
    let result: Vec<i32> = numbers
        .iter()
        .map(|x| x * 2)
        .filter(|x| x % 3 == 0)
        .collect();
    
    println!("Result: {:?}", result);
}
```

## Best Practices

1. **Prefer iterator methods** over manual loops for clarity and performance
2. **Chain operations** to avoid intermediate collections
3. **Use appropriate adapters** for the task (map, filter, fold, etc.)
4. **Leverage lazy evaluation** for performance with large datasets
5. **Implement Iterator trait** for custom data structures
6. **Use collect() wisely** - specify the collection type when needed

## Common Pitfalls

### Consuming Iterators Multiple Times

```rust
fn consuming_multiple_times() {
    let numbers = vec![1, 2, 3];
    let iter = numbers.iter();
    
    // First consumption
    let sum: i32 = iter.clone().sum();  // Use clone() if you need to reuse
    println!("Sum: {}", sum);
    
    // Second consumption (would error without clone)
    let count = iter.count();
    println!("Count: {}", count);
}
```

### Infinite Iterators

```rust
fn infinite_iterator_pitfall() {
    let natural_numbers = 1..;
    
    // This would run forever without take()
    // let all_numbers: Vec<i32> = natural_numbers.collect();
    
    // Correct: use take() to limit
    let first_ten: Vec<i32> = natural_numbers.take(10).collect();
    println!("First ten: {:?}", first_ten);
}
```

## Summary

Rust iterators provide:

- **Zero-cost abstractions** that compile to efficient code
- **Functional programming patterns** with map, filter, fold, etc.
- **Lazy evaluation** for performance with large datasets
- **Type safety** through the type system
- **Clean, readable code** compared to manual loops

Mastering iterators is essential for writing idiomatic, efficient Rust code, especially when working with collections and data processing pipelines.