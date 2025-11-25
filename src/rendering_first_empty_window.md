# Three-d Setup

Set up three-d and open a blank window.

## Dependencies (`Cargo.toml`)

```toml
[dependencies]
three-d = { version = "0.18.2", features = ["window"] }

[dev-dependencies]
cargo-watch = "8" # optional: rebuild/reload when meshes change
```

> Tip: replace `0.18.2` with the latest version of `three-d` if a newer one is available.

## Minimal app

```rust
use three_d::*;

fn main() {
    // Create a window + GL/WGPU context managed by three-d.
    let window = Window::new(WindowSettings {
        title: "Truck + three-d".into(),
        ..Default::default()
    })
    .unwrap();
    let _context = window.gl(); // grab GL context (currently unused)

    let mut camera = Camera::new_perspective(
        window.viewport(),
        vec3(3.0, 3.0, 3.0), // eye
        vec3(0.0, 0.0, 0.0), // target
        vec3(0.0, 1.0, 0.0), // up
        degrees(55.0),
        0.1,
        100.0,
    );

    window.render_loop(move |frame_input| {
        camera.set_viewport(frame_input.viewport);
        let screen = frame_input.screen();

        // Clear only; later sections will draw meshes.
        screen.clear(ClearState::color_and_depth(
            0.05, 0.05, 0.08, 1.0, 1.0,
        ));

        FrameOutput::default()
    });
}
```

## Run it

```bash
cargo run
```

You should see an empty dark window. If it opens, three-d is ready for rendering Truck meshes.
