# async main, mutex

## cargo.toml

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
futures = "0.3"
```

## code

```rust
use tokio::sync::Mutex;
use futures::executor::block_on;

fn main() {
    block_on(lock_func());
}

async fn lock_func() {
    let lock = Mutex::new(0);
    let mut a = lock.lock().await;
    *a += 10;
    println!("{}", *a);
}
```