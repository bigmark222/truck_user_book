# Viewer: Rendering With Bevy

We’ll use [`Bevy`](https://crates.io/crates/bevy) to open windows, load OBJ files exported from Truck, and preview them quickly. Bevy and Rapier will live together in a single crate (`truck_viewer`), so the rendering setup here becomes the foundation for physics in Chapter 5.

Topics:

- **4.1 Bevy Setup** — add Bevy and open a blank window
- **4.2 Render a Mesh** — load and render a Truck OBJ with lights/camera
- **4.3 Orbit + Hot Reload** — navigation and auto-refresh on file change
- **4.4 Reusable Viewer** — “drop in an OBJ and inspect” tool


## Project Layout

**From** `truck_workspace/` (parent directory of `truck_brep` and `truck_modeling`) **run**:
```sh
cargo new --lib truck_viewer
```

<br>

`truck_workspace/` should now look like:

```
truck-workspace/
├─ truck_meshes/   # Chapter 2 mesh crate (unchanged)
└─ truck_brep/     # Chapter 3 B-rep crate you’ll create below
└─ truck_viewer/  # Chapter 4 & 5 viewing obj and bringing it to life.
```


We want our truck_viewer directory to look like:

```
truck_viewer/
├── Cargo.toml
└── src/
    ├── lib.rs        # exports build_app/run_with_mesh and sync types
    ├── main.rs       # tiny binary to run the viewer standalone
    ├── app.rs        # sets up plugins, cameras, lights
    └── sync.rs       # sync editor state <-> bevy world
```

<br> 


So from our parent directory (`truck_workspace`) let's run:

```sh
touch truck_viewer/src/{main.rs,app.rs,sync.rs}
```

to create `main.rs`, `app.rs` and `sync.rs`. (`lib.rs` already exists from running `cargo new --lib`)
