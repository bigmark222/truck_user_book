# Rendering

We’ll use the Rust crate [`Bevy`](https://crates.io/crates/bevy) to open windows, load OBJ files exported from Truck, and preview them quickly.

Topics:

- **4.1 Bevy Setup** — add Bevy and open a blank window
- **4.2 Render a Mesh** — load and render a Truck OBJ with lights/camera
- **4.3 Orbit + Hot Reload** — navigation and auto-refresh on file change
- **4.4 Reusable Viewer** — “drop in an OBJ and inspect” tool

We’ll also use `cargo watch` to automatically reload whenever the OBJ file you’re working on is updated.
