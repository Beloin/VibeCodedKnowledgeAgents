# Rust Smart Pointers - Comprehensive Guide

## Introduction to Smart Pointers

Smart pointers are data structures that act like pointers but also have additional metadata and capabilities. In Rust, smart pointers own the data they point to, unlike references which only borrow data.

## Box<T> - Heap Allocation

### Basic Usage

```rust
fn box_basics() {
    // Store a value on the heap
    let b = Box::new(5);
    println!("b = {}", b);
    
    // Box automatically dereferences
    let x = *b + 1;
    println!("x = {}", x);
}
```

### Recursive Types

Box is essential for recursive data structures:

```rust
// Without Box, this wouldn't compile because the size isn't known at compile time
#[derive(Debug)]
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn recursive_list() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil)))));
    println!("List: {:?}", list);
}
```

### Binary Tree Example

```rust
type Link<T> = Option<Box<Node<T>>>;

struct Node<T> {
    value: T,
    left: Link<T>,
    right: Link<T>,
}

impl<T> Node<T> {
    fn new(value: T) -> Self {
        Self {
            value,
            left: None,
            right: None,
        }
    }
    
    fn set_left(&mut self, node: Node<T>) {
        self.left = Some(Box::new(node));
    }
    
    fn set_right(&mut self, node: Node<T>) {
        self.right = Some(Box::new(node));
    }
}

fn binary_tree_example() {
    let mut root = Node::new(1);
    root.set_left(Node::new(2));
    root.set_right(Node::new(3));
    
    // The tree structure is stored on the heap
}
```

## Rc<T> - Reference Counting

### Multiple Ownership

```rust
use std::rc::Rc;

fn rc_basics() {
    let a = Rc::new(String::from("hello"));
    println!("Reference count: {}", Rc::strong_count(&a));  // 1
    
    {
        let b = Rc::clone(&a);
        println!("Reference count: {}", Rc::strong_count(&a));  // 2
        
        let c = Rc::clone(&a);
        println!("Reference count: {}", Rc::strong_count(&a));  // 3
    } // b and c go out of scope
    
    println!("Reference count: {}", Rc::strong_count(&a));  // 1
}
```

### Graph Structure

```rust
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    edges: Vec<Rc<Node>>,
}

impl Node {
    fn new(value: i32) -> Rc<Self> {
        Rc::new(Self {
            value,
            edges: Vec::new(),
        })
    }
    
    fn add_edge(&mut self, node: Rc<Node>) {
        self.edges.push(node);
    }
}

fn graph_example() {
    let node1 = Node::new(1);
    let node2 = Node::new(2);
    let node3 = Node::new(3);
    
    // We need to get mutable references to add edges
    Rc::get_mut(&mut node1.clone()).unwrap().add_edge(Rc::clone(&node2));
    Rc::get_mut(&mut node1.clone()).unwrap().add_edge(Rc::clone(&node3));
    
    println!("Node1: {:?}", node1);
}
```

### Weak References

```rust
use std::rc::{Rc, Weak};

struct TreeNode {
    value: i32,
    parent: Option<Weak<TreeNode>>,
    children: Vec<Rc<TreeNode>>,
}

impl TreeNode {
    fn new(value: i32) -> Rc<Self> {
        Rc::new(Self {
            value,
            parent: None,
            children: Vec::new(),
        })
    }
    
    fn add_child(parent: &Rc<Self>, value: i32) -> Rc<Self> {
        let child = Self::new(value);
        child.parent = Some(Rc::downgrade(parent));
        Rc::get_mut(parent).unwrap().children.push(Rc::clone(&child));
        child
    }
}

fn tree_with_weak_refs() {
    let root = TreeNode::new(0);
    let child1 = TreeNode::add_child(&root, 1);
    let child2 = TreeNode::add_child(&root, 2);
    
    // Check parent reference
    if let Some(parent) = child1.parent.as_ref().and_then(|w| w.upgrade()) {
        println!("Child1's parent value: {}", parent.value);
    }
}
```

## Arc<T> - Atomic Reference Counting

### Thread-Safe Reference Counting

```rust
use std::sync::Arc;
use std::thread;

fn arc_basics() {
    let counter = Arc::new(0);
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            // Each thread gets its own Arc
            let _local_counter = counter;
            // The counter will be dropped when the thread ends
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Final reference count: {}", Arc::strong_count(&counter));
}
```

### Shared Configuration

```rust
use std::sync::Arc;
use std::thread;

#[derive(Debug)]
struct Config {
    database_url: String,
    max_connections: u32,
}

impl Config {
    fn new() -> Self {
        Self {
            database_url: "postgres://localhost:5432".to_string(),
            max_connections: 10,
        }
    }
}

fn shared_config_example() {
    let config = Arc::new(Config::new());
    let mut handles = vec![];
    
    for i in 0..5 {
        let config = Arc::clone(&config);
        let handle = thread::spawn(move || {
            println!("Thread {} using config: {:?}", i, config);
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

## RefCell<T> - Interior Mutability

### Runtime Borrow Checking

```rust
use std::cell::RefCell;

fn refcell_basics() {
    let x = RefCell::new(42);
    
    {
        let mut y = x.borrow_mut();
        *y += 1;
    } // y goes out of scope, borrow ends
    
    let z = x.borrow();
    println!("Value: {}", *z);
    
    // This would panic at runtime:
    // let mut y = x.borrow_mut();
    // let z = x.borrow(); // Panic: already borrowed mutably
}
```

### Mock Objects for Testing

```rust
use std::cell::RefCell;

pub trait Messenger {
    fn send(&self, msg: &str);
}

struct MockMessenger {
    sent_messages: RefCell<Vec<String>>,
}

impl MockMessenger {
    fn new() -> Self {
        Self {
            sent_messages: RefCell::new(vec![]),
        }
    }
}

impl Messenger for MockMessenger {
    fn send(&self, msg: &str) {
        self.sent_messages.borrow_mut().push(msg.to_string());
    }
}

fn test_messenger() {
    let mock = MockMessenger::new();
    mock.send("hello");
    mock.send("world");
    
    assert_eq!(mock.sent_messages.borrow().len(), 2);
    assert_eq!(mock.sent_messages.borrow()[0], "hello");
}
```

### Cache with Interior Mutability

```rust
use std::cell::RefCell;
use std::collections::HashMap;

struct Cache {
    data: RefCell<HashMap<String, String>>,
}

impl Cache {
    fn new() -> Self {
        Self {
            data: RefCell::new(HashMap::new()),
        }
    }
    
    fn get(&self, key: &str) -> Option<String> {
        self.data.borrow().get(key).cloned()
    }
    
    fn set(&self, key: &str, value: &str) {
        self.data.borrow_mut().insert(key.to_string(), value.to_string());
    }
    
    fn clear(&self) {
        self.data.borrow_mut().clear();
    }
}

fn cache_example() {
    let cache = Cache::new();
    cache.set("key1", "value1");
    cache.set("key2", "value2");
    
    println!("Key1: {:?}", cache.get("key1"));
    println!("Key2: {:?}", cache.get("key2"));
    
    cache.clear();
    println!("After clear: {:?}", cache.get("key1"));
}
```

## Combining Smart Pointers

### Rc + RefCell Pattern

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}

impl Node {
    fn new(value: i32) -> Rc<Self> {
        Rc::new(Self {
            value,
            children: RefCell::new(Vec::new()),
        })
    }
    
    fn add_child(parent: &Rc<Self>, child: &Rc<Self>) {
        parent.children.borrow_mut().push(Rc::clone(child));
    }
}

fn tree_with_mutability() {
    let root = Node::new(0);
    let child1 = Node::new(1);
    let child2 = Node::new(2);
    
    Node::add_child(&root, &child1);
    Node::add_child(&root, &child2);
    
    println!("Root: {:?}", root);
}
```

### Arc + Mutex for Thread-Safe Mutability

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn thread_safe_counter() {
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
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Result: {}", *counter.lock().unwrap());
}
```

## Custom Smart Pointers

### Implementing Deref

```rust
use std::ops::{Deref, DerefMut};

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> Self {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;
    
    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

impl<T> DerefMut for MyBox<T> {
    fn deref_mut(&mut self) -> &mut Self::Target {
        &mut self.0
    }
}

fn custom_box_example() {
    let x = 5;
    let y = MyBox::new(x);
    
    assert_eq!(5, x);
    assert_eq!(5, *y);  // Deref coercion
    
    let mut z = MyBox::new(String::from("hello"));
    z.push_str(", world!");  // DerefMut allows mutable operations
    println!("{}", *z);
}
```

### Drop Trait Implementation

```rust
struct CustomSmartPointer {
    data: String,
}

impl CustomSmartPointer {
    fn new(data: &str) -> Self {
        println!("Creating CustomSmartPointer with data: {}", data);
        Self {
            data: data.to_string(),
        }
    }
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data: {}", self.data);
    }
}

fn drop_example() {
    let c = CustomSmartPointer::new("my stuff");
    let d = CustomSmartPointer::new("other stuff");
    println!("CustomSmartPointers created.");
    // c and d go out of scope here, drop is called automatically
}

fn early_drop_example() {
    let c = CustomSmartPointer::new("some data");
    println!("CustomSmartPointer created.");
    drop(c);  // Explicitly drop before scope ends
    println!("CustomSmartPointer dropped before the end of main.");
}
```

## Real-World Examples

### Observer Pattern

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

pub trait Observer {
    fn update(&self, message: &str);
}

pub struct Subject {
    observers: RefCell<Vec<Weak<dyn Observer>>>,
}

impl Subject {
    pub fn new() -> Self {
        Self {
            observers: RefCell::new(Vec::new()),
        }
    }
    
    pub fn attach(&self, observer: Weak<dyn Observer>) {
        self.observers.borrow_mut().push(observer);
    }
    
    pub fn notify(&self, message: &str) {
        let mut observers = self.observers.borrow_mut();
        observers.retain(|observer| {
            if let Some(observer) = observer.upgrade() {
                observer.update(message);
                true
            } else {
                false
            }
        });
    }
}

struct ConcreteObserver {
    name: String,
}

impl ConcreteObserver {
    fn new(name: &str) -> Rc<Self> {
        Rc::new(Self {
            name: name.to_string(),
        })
    }
}

impl Observer for ConcreteObserver {
    fn update(&self, message: &str) {
        println!("Observer {} received: {}", self.name, message);
    }
}

fn observer_example() {
    let subject = Subject::new();
    
    let observer1 = ConcreteObserver::new("Observer1");
    let observer2 = ConcreteObserver::new("Observer2");
    
    subject.attach(Rc::downgrade(&observer1));
    subject.attach(Rc::downgrade(&observer2));
    
    subject.notify("Hello, observers!");
}
```

### Resource Pool

```rust
use std::sync::{Arc, Mutex};
use std::collections::VecDeque;

struct Connection {
    id: u32,
}

impl Connection {
    fn new(id: u32) -> Self {
        println!("Creating connection {}", id);
        Self { id }
    }
    
    fn execute_query(&self, query: &str) {
        println!("Connection {} executing: {}", self.id, query);
    }
}

impl Drop for Connection {
    fn drop(&mut self) {
        println!("Dropping connection {}", self.id);
    }
}

struct ConnectionPool {
    connections: Arc<Mutex<VecDeque<Connection>>>,
    next_id: Arc<Mutex<u32>>,
}

impl ConnectionPool {
    fn new() -> Self {
        Self {
            connections: Arc::new(Mutex::new(VecDeque::new())),
            next_id: Arc::new(Mutex::new(0)),
        }
    }
    
    fn get_connection(&self) -> PooledConnection {
        let mut connections = self.connections.lock().unwrap();
        
        if let Some(conn) = connections.pop_front() {
            PooledConnection {
                connection: Some(conn),
                pool: Arc::clone(&self.connections),
            }
        } else {
            let mut next_id = self.next_id.lock().unwrap();
            let conn = Connection::new(*next_id);
            *next_id += 1;
            
            PooledConnection {
                connection: Some(conn),
                pool: Arc::clone(&self.connections),
            }
        }
    }
}

struct PooledConnection {
    connection: Option<Connection>,
    pool: Arc<Mutex<VecDeque<Connection>>>,
}

impl PooledConnection {
    fn execute_query(&self, query: &str) {
        if let Some(conn) = &self.connection {
            conn.execute_query(query);
        }
    }
}

impl Drop for PooledConnection {
    fn drop(&mut self) {
        if let Some(conn) = self.connection.take() {
            let mut pool = self.pool.lock().unwrap();
            pool.push_back(conn);
        }
    }
}

fn connection_pool_example() {
    let pool = ConnectionPool::new();
    
    {
        let conn1 = pool.get_connection();
        conn1.execute_query("SELECT * FROM users");
        
        let conn2 = pool.get_connection();
        conn2.execute_query("SELECT * FROM products");
        
        // Connections are returned to pool when they go out of scope
    }
    
    // Reuse connections from pool
    let conn3 = pool.get_connection();
    conn3.execute_query("SELECT * FROM orders");
}
```

## Performance Considerations

- **Box<T>**: Minimal overhead, just heap allocation
- **Rc<T>**: Reference counting overhead, not thread-safe
- **Arc<T>**: Atomic reference counting, thread-safe but more expensive
- **RefCell<T>**: Runtime borrow checking, no atomic operations
- **Combinations**: Rc + RefCell has both reference counting and runtime borrow checking overhead

## Best Practices

1. **Use Box for recursive types** and when you need heap allocation
2. **Use Rc for multiple ownership** within a single thread
3. **Use Arc for multiple ownership** across threads
4. **Use RefCell for interior mutability** when you need to mutate through shared references
5. **Avoid circular references** with Rc by using Weak references
6. **Prefer compile-time checks** over runtime checks when possible

## Summary

Smart pointers in Rust provide:

- **Memory safety** through ownership and borrowing rules
- **Flexible memory management** with different ownership models
- **Thread safety** with atomic reference counting
- **Interior mutability** patterns for shared mutation
- **Zero-cost abstractions** for performance-critical code

Mastering smart pointers is essential for building complex data structures and concurrent applications in Rust.