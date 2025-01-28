# mini-redis

A minimal Redis client and server implementation using Tokio - designed for learning asynchronous Rust programming.

## Project Overview

This project is an educational implementation of a Redis-like key-value store, focusing on demonstrating core concepts of asynchronous programming in Rust using the Tokio runtime. It includes a basic server handling GET/SET commands and a client showcasing concurrent requests using Tokio's concurrency primitives.

## Features

- **Server**:
  - Async TCP server using `tokio::net`
  - In-memory key-value store protected by `Mutex`
  - Supports basic commands:
    - `GET <key>`
    - `SET <key> <value>`
- **Client**:
  - Connection management with async/await
  - Concurrent request handling using:
    - Multi-producer single-consumer (MPSC) channels
    - Oneshot channels for response propagation
  - Example usage with concurrent GET/SET operations


## Installation

   ```bash
   git clone https://github.com/yourusername/mini-redis.git
   cd mini-redis
   cargo build
   ```

## Usage

1. **Start the Server**:
   ```bash
   cargo run --bin server
   ```
   Listens on `127.0.0.1:6379`

2. **Run Example Client (on another terminal)**:
   ```bash
   cargo run --example hello-redis
   ```
   Sets "Hello" = "World" and retrieves it

3. **Concurrent Client Demo**:
   ```bash
   cargo run --bin client
   ```
   Demonstrates concurrent GET/SET operations using Tokio tasks

## Code Examples

### Basic Usage (`examples/hello-redis.rs`)
```rust
use mini_redis::{client, Result};

#[tokio::main]
async fn main() -> Result<()> {
    let mut client = client::connect("127.0.0.1:6379").await?;
    client.set("Hello", "World".into()).await?;
    let result = client.get("Hello").await?;
    println!("Got value from server: {:?}", result);
    Ok(())
}
```

### Concurrent Client Implementation Highlights
- Manager task handles connection pooling
- MPSC channel for command dispatch
- Oneshot channels for response handling
```rust
// Simplified structure showing concurrent SET/GET
let (tx, mut rx) = mpsc::channel(32);
let manager = tokio::spawn(async move {
    while let Some(cmd) = rx.recv().await {
        match cmd {
            Command::Get { key, resp } => {
                let res = client.get(&key).await;
                let _ = resp.send(res);
            }
            Command::Set { key, val, resp } => {
                let res = client.set(&key, val).await;
                let _ = resp.send(res);
            }
        }
    }
});
```

## Project Structure
```
mini-redis/
├── src/
│   ├── lib.rs          # Core client/server implementation
│   └── bin/
│       ├── client.rs   # Concurrent client demo
│       └── server.rs   # Key-value store server
├── examples/
│   └── hello-redis.rs  # Basic usage example
└── Cargo.toml
```
