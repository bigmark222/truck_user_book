# Orbit + Hot Reload

Add basic orbit controls and make the viewer reload whenever the OBJ file changes.

## Accept a mesh path

Update `main` to read a file path (defaulting to `output/mesh.obj`):

```rust
let path = std::env::args().nth(1).unwrap_or_else(|| "output/mesh.obj".into());
let mut cpu_mesh = load_mesh(&path); // reuse the loader from the simple viewer section
```

Now you can run `cargo run -- output/mesh.obj` or point at any other OBJ generated earlier.

## Orbit + zoom

Add simple mouse controls inside `render_loop`:

```rust
let mut yaw = 0.0f32;
let mut pitch = 0.0f32;
let mut distance = 6.0f32;

window.render_loop(move |mut frame_input| {
    // Mouse drag to orbit (read your preferred delta API from FrameInput)
    if let Some(delta) = frame_input.mouse_motion() {
        yaw += delta.x * 0.01;
        pitch = (pitch + delta.y * 0.01).clamp(-1.2, 1.2);
    }

    // Scroll to zoom
    if let Some(scroll) = frame_input.scroll_delta() {
        distance = (distance - scroll.y * 0.3).clamp(2.0, 20.0);
    }

    // Update camera from spherical coords
    let dir = vec3(
        yaw.sin() * pitch.cos(),
        pitch.sin(),
        yaw.cos() * pitch.cos(),
    );
    camera.set_view(
        vec3(0.0, 0.0, 0.0) + dir * distance,
        vec3(0.0, 0.0, 0.0),
        vec3(0.0, 1.0, 0.0),
    );

    // ... render code from previous section ...
```

The exact mouse/scroll helpers depend on your three-d version; any delta events will work as long as you update `yaw`, `pitch`, and `distance` similarly.

## Auto-reload with `cargo watch`

Install once:

```bash
cargo install cargo-watch
```

Run the viewer and rerun on file changes:

```bash
cargo watch -w output/mesh.obj -x 'run -- output/mesh.obj'
```

- `-w output/mesh.obj` watches the OBJ written by your Truck examples.
- When the file is replaced (e.g., you run another example that overwrites it), `cargo watch` restarts your viewer so the new mesh loads immediately.

You now have a lightweight orbit viewer that refreshes whenever your generated mesh changes.
