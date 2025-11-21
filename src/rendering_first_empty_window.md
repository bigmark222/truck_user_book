# First Empty Window

Set up Truck’s rendering environment and open a blank window (wgpu + winit).

Reference: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter4/src/section4_1.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter4/src/section4_1.rs)

## Dependencies (`Cargo.toml`)

```toml
[dependencies]
async-trait = "0.1.73"      # async trait support
winit = "0.28.6"            # window + event loop
truck-platform = "0.6.0"    # wgpu wrapper
truck-rendimpl = "0.6.0"    # shape/mesh rendering

[target.'cfg(not(target_arch = "wasm32"))'.dependencies]
instant = { version = "0.1.12", features = ["now"] }
pollster = "0.3.0"          # async GPU init helper

[target.'cfg(target_arch = "wasm32")'.dependencies]
instant = { version = "0.1.12", features = ["now", "wasm-bindgen"] }
wasm-bindgen-futures = "0.4.37"
```

## Add the helper module

Download `app.rs` from:
[github.com/ricosjp/truck/blob/truck-rendimpl-v0.6.0/truck-rendimpl/examples/app.rs](https://github.com/ricosjp/truck/blob/truck-rendimpl-v0.6.0/truck-rendimpl/examples/app.rs)

Place it at `src/app.rs`. It wraps rendering boilerplate and defines the `App` trait.

## Minimal app

```rust
mod app;
use app::*;

use std::sync::Arc;
use winit::window::Window;

struct MyApp;

// Use async_trait without Send because our renderer runs on one thread.
// Prevents Rust from requiring all async futures to be thread-safe.
#[async_trait(?Send)]
impl App for MyApp {
    async fn init(_: Arc<Window>) -> Self {
        MyApp
    }
}

fn main() {
    MyApp::run()
}
```

## Run it

```bash
cargo run
```

You should see a blank window with a black background—your Truck rendering setup is working.
