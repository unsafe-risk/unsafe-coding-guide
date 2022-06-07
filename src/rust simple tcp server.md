# rust simple TCP server

```rust
use std::{net::TcpListener, io::Read};

fn main() {
    let listener = TcpListener::bind("0.0.0.0:3333").expect("Could not bind to port");

    for stream in listener.incoming() {
        match stream {
            Ok(mut stream) => {
                println!("Connection established!");
                let mut total_buffer = String::new();
                loop {
                    let mut buffer = [0; 64];
                    match stream.read(&mut buffer) {
                        Ok(size) => {
                            total_buffer.push_str(&String::from_utf8_lossy(&buffer[..size]));
                            if size < 64 {
                                break;
                            }
                        }
                        Err(e) => {
                            println!("Error: {}", e);
                            break;
                        }
                    }
                }
                println!("{}", total_buffer);
            }
            Err(e) => {
                println!("Connection failed: {}", e);
            }
        }
    }
}
```

#tcp #server #rust