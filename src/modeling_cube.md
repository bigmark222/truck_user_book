# Cube

Model a precise B-rep cube with Truck—exact geometry built from vertices, edges, faces, and a solid.

## Dependencies

Add to `Cargo.toml`:

```toml
[dependencies]
truck-modeling = "0.6.0"   # B-rep modeling API
truck-meshalgo = "0.6.0"   # Mesh conversion utilities
truck-stepio  = "0.2.0"    # STEP export
```

## B-rep building blocks

From `truck_modeling::topology`:

- Vertex: point
- Edge: curve between two vertices
- Wire: sequence of edges forming a loop
- Face: bounded by a closed wire
- Shell: group of faces
- Solid: closed shell forming a volume

## Build a cube with `tsweep` (shapes module)

Group B-rep shapes under `src/shapes/`, re-export them from `lib.rs`, and keep `src/bin/cube.rs` tiny. `tsweep` sweeps a geometric element along a vector to produce the next higher element; four sweeps make a cube.

`src/shapes/cube.rs`:

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};
```

```rust

/// Build a solid cube by chaining translational sweeps.
pub fn cube() -> Solid {
    let vertex: Vertex = builder::vertex(Point3::new(-1.0, 0.0, -1.0));
    let edge:   Edge   = builder::tsweep(&vertex, 2.0 * Vector3::unit_z());
    let face:   Face   = builder::tsweep(&edge,   2.0 * Vector3::unit_x());
    builder::tsweep(&face, 2.0 * Vector3::unit_y())
}
```

<details>
<summary>Steps inside `cube()`:</summary>

1. Place the first vertex at `(-1, 0, -1)`.
2. Sweep 2 units along +Z → edge.
3. Sweep edge 2 units along +X → rectangular face.
4. Sweep face 2 units along +Y → solid cube.
</details>


```rust
/// Triangulate and write an OBJ mesh for quick viewing.
pub fn save_obj(solid: &Solid, path: &str) {
    let mesh_with_topology = solid.triangulation(0.01); // tolerance
    let mesh = mesh_with_topology.to_polygon();
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(&mesh, &mut obj).unwrap();
}
```

```rust
/// Export an exact STEP B-rep.
pub fn save_step(solid: &Solid, path: &str) {
    let compressed = solid.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(path, display.to_string()).unwrap();
}
```

## Expose the cube module through lib.rs

**src/lib.rs**

```rust
pub mod shapes;
pub use shapes::{cube, save_obj, save_step};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub use cube::{cube, save_obj, save_step};
```


## Main entry point (bin)

`src/bin/cube.rs` just calls the library functions:

```rust
fn main() {
    let cube = truck_brep::cube();
    truck_brep::save_obj(&cube, "output/cube.obj");
    truck_brep::save_step(&cube, "output/cube.step");
}
```

This keeps the binary minimal and lets other sections reuse the same helpers.

<details>
<summary>Directory tree</summary>

```
truck_brep/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs              # re-exports shapes
│  ├─ shapes/
│  │  ├─ mod.rs
│  │  └─ cube.rs          # cube(), save_obj, save_step
│  └─ bin/
│     └─ cube.rs          # calls into lib
└─ output/                # cube.obj, cube.step
```

</details>

</details>

<details>
<summary>Complete code (lib, shapes, bin)</summary>

**src/lib.rs**

```rust
pub mod shapes;
pub use shapes::{cube, save_obj, save_step};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub use cube::{cube, save_obj, save_step};
```

**src/shapes/cube.rs**

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

pub fn cube() -> Solid {
    let vertex: Vertex = builder::vertex(Point3::new(-1.0, 0.0, -1.0));
    let edge: Edge = builder::tsweep(&vertex, 2.0 * Vector3::unit_z());
    let face: Face = builder::tsweep(&edge, 2.0 * Vector3::unit_x());
    builder::tsweep(&face, 2.0 * Vector3::unit_y())
}

pub fn save_obj(solid: &Solid, path: &str) {
    let mesh_with_topology = solid.triangulation(0.01);
    let mesh = mesh_with_topology.to_polygon();
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(&mesh, &mut obj).unwrap();
}

pub fn save_step(solid: &Solid, path: &str) {
    let compressed = solid.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(path, display.to_string()).unwrap();
}
```

**src/bin/cube.rs**

```rust
fn main() {
    let cube = truck_brep::cube();
    truck_brep::save_obj(&cube, "output/cube.obj");
    truck_brep::save_step(&cube, "output/cube.step");
}
```

</details>
