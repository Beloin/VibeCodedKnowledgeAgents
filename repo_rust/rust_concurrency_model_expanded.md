# Rust Concurrency Model - Comprehensive Guide

## Introduction to Fearless Concurrency

Rust's concurrency model is designed to prevent data races at compile time through its ownership system. This "fearless concurrency" approach allows you to write safe concurrent code without runtime overhead.

## Threads

### Basic Thread Creation

```rust
use std::thread;
use std::time::Duration;

fn basic_threads() {
    // Spawn a new thread
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });
    
    // Main thread continues executing
    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
    
    // Wait for the spawned thread to finish
    handle.join().unwrap();
}
```

### Threads with Move Closures

```rust
fn threads_with_move() {
    let v = vec![1, 2, 3];
    
    // Use move to transfer ownership to the thread
    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
        // v is moved into the closure
    });
    
    // v is no longer valid here
    // println!("{:?}", v);  // Error: value moved
    
    handle.join().unwrap();
}
```

### Multiple Threads

```rust
fn multiple_threads() {
    let mut handles = vec![];
    
    for i in 0..10 {
        let handle = thread::spawn(move || {
            println!("Thread {} starting", i);
            thread::sleep(Duration::from_millis(100));
            println!("Thread {} finished", i);
        });
        handles.push(handle);
    }
    
    // Wait for all threads to complete
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("All threads completed!");
}
```

## Message Passing

### Basic Channels

```rust
use std::sync::mpsc;
use std::thread;

fn basic_channels() {
    // Create a channel
    let (tx, rx) = mpsc::channel();
    
    // Spawn a thread to send messages
    thread::spawn(move || {
        let val = String::from("hello");
        tx.send(val).unwrap();
        // val is moved, can't use it here
    });
    
    // Receive the message
    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

### Multiple Producers

```rust
fn multiple_producers() {
    let (tx, rx) = mpsc::channel();
    
    // Clone the transmitter for multiple producers
    let tx1 = tx.clone();
    let tx2 = tx.clone();
    
    // First producer
    thread::spawn(move || {
        let messages = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];
        
        for msg in messages {
            tx1.send(msg).unwrap();
            thread::sleep(Duration::from_millis(100));
        }
    });
    
    // Second producer
    thread::spawn(move || {
        let messages = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];
        
        for msg in messages {
            tx2.send(msg).unwrap();
            thread::sleep(Duration::from_millis(150));
        }
    });
    
    // Receive all messages
    for received in rx {
        println!("Got: {}", received);
    }
}
```

### Channel with Timeouts

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn channel_with_timeout() {
    let (tx, rx) = mpsc::channel();
    
    thread::spawn(move || {
        thread::sleep(Duration::from_millis(500));
        tx.send("delayed message").unwrap();
    });
    
    // Try to receive with timeout
    match rx.recv_timeout(Duration::from_millis(100)) {
        Ok(msg) => println!("Got: {}", msg),
        Err(mpsc::RecvTimeoutError::Timeout) => println!("Timeout!"),
        Err(mpsc::RecvTimeoutError::Disconnected) => println!("Channel disconnected!"),
    }
}
```

## Shared State Concurrency

### Mutex Basics

```rust
use std::sync::Mutex;

fn mutex_basics() {
    let m = Mutex::new(5);
    
    {
        let mut num = m.lock().unwrap();
        *num = 6;
    } // Lock is automatically released here
    
    println!("m = {:?}", m);
}
```

### Arc + Mutex Pattern

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn arc_mutex_counter() {
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

### RwLock for Read-Write Locks

```rust
use std::sync::{Arc, RwLock};
use std::thread;

fn rwlock_example() {
    let data = Arc::new(RwLock::new(0));
    let mut handles = vec![];
    
    // Multiple readers
    for i in 0..5 {
        let data = Arc::clone(&data);
        let handle = thread::spawn(move || {
            let reader = data.read().unwrap();
            println!("Reader {}: {}", i, *reader);
        });
        handles.push(handle);
    }
    
    // One writer
    let data_writer = Arc::clone(&data);
    let writer_handle = thread::spawn(move || {
        let mut writer = data_writer.write().unwrap();
        *writer += 1;
        println!("Writer: updated to {}", *writer);
    });
    handles.push(writer_handle);
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

## Advanced Concurrency Patterns

### Thread Pool

```rust
use std::sync::{Arc, Mutex, mpsc};
use std::thread;

struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
        
        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));
        
        let mut workers = Vec::with_capacity(size);
        
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
        
        ThreadPool { workers, sender }
    }
    
    fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);
        self.sender.send(job).unwrap();
    }
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();
            println!("Worker {} got a job; executing.", id);
            job();
        });
        
        Worker { id, thread }
    }
}

fn thread_pool_example() {
    let pool = ThreadPool::new(4);
    
    for i in 0..8 {
        pool.execute(move || {
            println!("Executing task {}", i);
            thread::sleep(Duration::from_millis(500));
        });
    }
    
    // Give threads time to complete
    thread::sleep(Duration::from_secs(2));
}
```

### Atomic Operations

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn atomic_counter() {
    let counter = Arc::new(AtomicUsize::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 0..1000 {
                counter.fetch_add(1, Ordering::SeqCst);
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Result: {}", counter.load(Ordering::SeqCst));
}
```

### Barrier Synchronization

```rust
use std::sync::{Arc, Barrier};
use std::thread;

fn barrier_example() {
    let barrier = Arc::new(Barrier::new(3));
    let mut handles = vec![];
    
    for i in 0..3 {
        let barrier = Arc::clone(&barrier);
        let handle = thread::spawn(move || {
            println!("Thread {} before barrier", i);
            barrier.wait();
            println!("Thread {} after barrier", i);
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

## Real-World Examples

### Web Server with Thread Pool

```rust
use std::net::{TcpListener, TcpStream};
use std::io::prelude::*;
use std::sync::{Arc, Mutex, mpsc};
use std::thread;

struct WebServer {
    pool: ThreadPool,
}

impl WebServer {
    fn new(pool_size: usize) -> Self {
        Self {
            pool: ThreadPool::new(pool_size),
        }
    }
    
    fn handle_connection(mut stream: TcpStream) {
        let mut buffer = [0; 1024];
        stream.read(&mut buffer).unwrap();
        
        let response = "HTTP/1.1 200 OK\r\n\r\nHello, World!";
        stream.write(response.as_bytes()).unwrap();
        stream.flush().unwrap();
    }
    
    fn start(&self, address: &str) {
        let listener = TcpListener::bind(address).unwrap();
        println!("Server listening on {}", address);
        
        for stream in listener.incoming() {
            let stream = stream.unwrap();
            
            self.pool.execute(|| {
                WebServer::handle_connection(stream);
            });
        }
    }
}

// Note: This is a simplified example. Real web servers would be more complex.
```

### Producer-Consumer Pattern

```rust
use std::sync::{Arc, Mutex, mpsc};
use std::thread;
use std::time::Duration;

struct ProducerConsumer {
    buffer: Arc<Mutex<Vec<i32>>>,
    capacity: usize,
}

impl ProducerConsumer {
    fn new(capacity: usize) -> Self {
        Self {
            buffer: Arc::new(Mutex::new(Vec::with_capacity(capacity))),
            capacity,
        }
    }
    
    fn producer(&self, id: usize, count: usize) {
        for i in 0..count {
            let item = (id * 1000) + i as i32;
            
            loop {
                let mut buffer = self.buffer.lock().unwrap();
                if buffer.len() < self.capacity {
                    buffer.push(item);
                    println!("Producer {} produced: {}", id, item);
                    break;
                }
                // Buffer full, release lock and try again
                drop(buffer);
                thread::sleep(Duration::from_millis(10));
            }
            
            thread::sleep(Duration::from_millis(50));
        }
    }
    
    fn consumer(&self, id: usize) {
        loop {
            let item = {
                let mut buffer = self.buffer.lock().unwrap();
                if let Some(item) = buffer.pop() {
                    item
                } else {
                    // Buffer empty
                    continue;
                }
            };
            
            println!("Consumer {} consumed: {}", id, item);
            thread::sleep(Duration::from_millis(100));
        }
    }
}

fn producer_consumer_example() {
    let pc = Arc::new(ProducerConsumer::new(5));
    let mut handles = vec![];
    
    // Start producers
    for i in 0..2 {
        let pc = Arc::clone(&pc);
        let handle = thread::spawn(move || {
            pc.producer(i, 10);
        });
        handles.push(handle);
    }
    
    // Start consumers
    for i in 0..3 {
        let pc = Arc::clone(&pc);
        let handle = thread::spawn(move || {
            pc.consumer(i);
        });
        handles.push(handle);
    }
    
    // Wait for producers to finish
    for handle in handles.drain(0..2) {
        handle.join().unwrap();
    }
    
    // Give consumers time to finish processing
    thread::sleep(Duration::from_secs(2));
}
```

### Parallel Data Processing

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn parallel_data_processing() {
    let data = Arc::new(vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);
    let results = Arc::new(Mutex::new(Vec::new()));
    let mut handles = vec![];
    
    let chunk_size = data.len() / 4;
    
    for i in 0..4 {
        let data = Arc::clone(&data);
        let results = Arc::clone(&results);
        
        let handle = thread::spawn(move || {
            let start = i * chunk_size;
            let end = if i == 3 { data.len() } else { (i + 1) * chunk_size };
            
            let chunk = &data[start..end];
            let chunk_sum: i32 = chunk.iter().sum();
            
            let mut results = results.lock().unwrap();
            results.push(chunk_sum);
        });
        
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    let final_results = results.lock().unwrap();
    let total_sum: i32 = final_results.iter().sum();
    println!("Total sum: {}", total_sum);
}
```

## Performance Considerations

### Lock Granularity

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn fine_grained_locking() {
    // Instead of one big lock:
    // let data = Arc::new(Mutex::new((vec![], vec![])));
    
    // Use separate locks for independent data
    let data1 = Arc::new(Mutex::new(vec![]));
    let data2 = Arc::new(Mutex::new(vec![]));
    
    let mut handles = vec![];
    
    // Threads can work on data1 and data2 independently
    let data1_clone = Arc::clone(&data1);
    let handle1 = thread::spawn(move || {
        let mut data = data1_clone.lock().unwrap();
        data.push(1);
    });
    
    let data2_clone = Arc::clone(&data2);
    let handle2 = thread::spawn(move || {
        let mut data = data2_clone.lock().unwrap();
        data.push(2);
    });
    
    handles.push(handle1);
    handles.push(handle2);
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

### Avoiding Lock Contention

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn lock_free_counter() {
    let counter = Arc::new(AtomicUsize::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 0..1000 {
                // Atomic operations are lock-free
                counter.fetch_add(1, Ordering::Relaxed);
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Counter: {}", counter.load(Ordering::Relaxed));
}
```

## Best Practices

1. **Prefer message passing** over shared state when possible
2. **Use appropriate synchronization primitives** for your use case
3. **Keep critical sections small** to minimize lock contention
4. **Use atomic operations** for simple counters and flags
5. **Consider thread pools** for CPU-bound work
6. **Handle thread panics** gracefully
7. **Use RAII patterns** for resource management

## Common Pitfalls

### Deadlocks

```rust
fn deadlock_example() {
    let lock1 = Arc::new(Mutex::new(0));
    let lock2 = Arc::new(Mutex::new(0));
    
    let lock1_clone = Arc::clone(&lock1);
    let lock2_clone = Arc::clone(&lock2);
    
    let handle1 = thread::spawn(move || {
        let _guard1 = lock1_clone.lock().unwrap();
        thread::sleep(Duration::from_millis(10));
        let _guard2 = lock2_clone.lock().unwrap();  // Potential deadlock
    });
    
    let handle2 = thread::spawn(move || {
        let _guard2 = lock2.lock().unwrap();
        thread::sleep(Duration::from_millis(10));
        let _guard1 = lock1.lock().unwrap();  // Potential deadlock
    });
    
    handle1.join().unwrap();
    handle2.join().unwrap();
}
```

### Data Races (Prevented by Compiler)

```rust
fn data_race_prevention() {
    let mut data = vec![1, 2, 3];
    
    // This would cause a compile error:
    // thread::spawn(|| {
    //     data.push(4);  // Error: closure may outlive current function
    // });
    
    // Correct approach: move ownership
    let handle = thread::spawn(move || {
        let mut local_data = data;
        local_data.push(4);
        local_data
    });
    
    let result = handle.join().unwrap();
    println!("Result: {:?}", result);
}
```

## Summary

Rust's concurrency model provides:

- **Memory safety** through compile-time checks
- **Data race prevention** via ownership system
- **Zero-cost abstractions** for performance
- **Flexible synchronization** with channels, mutexes, and atomics
- **Fearless concurrency** that catches errors at compile time

Mastering Rust's concurrency features is essential for building high-performance, safe concurrent applications.