# Torus

Build a torus (donut) with rotational sweeps, keeping the geometry and exports in the library just like the cube section. For the cylinder example, see `modeling_cylinder.md`.

Reference: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter3/src/section3_2.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter3/src/section3_2.rs)

## Torus via rotational sweeps (shapes module)

Use `rsweep` twice: first to spin a vertex into a circle, then to spin that circle into the torus shell. Keep this in `src/shapes/torus.rs` so binaries stay minimal.

**src/shapes/torus.rs**

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

/// Build a torus shell with nested rotational sweeps.
pub fn torus() -> Shell {
    let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, 1.0));

    // Sweep the vertex around +X to form a circular wire
    let circle: Wire = builder::rsweep(
        &vertex,
        Point3::new(0.0, 0.5, 1.0), // point on rotation axis
        Vector3::unit_x(),         // axis direction
        Rad(7.0),                  // angle (~401°) to ensure closure
    );

    // Sweep the circle around +Y to form the torus shell
    builder::rsweep(&circle, Point3::origin(), Vector3::unit_y(), Rad(7.0))
}

/// Triangulate the shell and write an OBJ mesh.
pub fn save_torus_obj(shell: &Shell, path: &str) {
    let mesh_with_topology = shell.triangulation(0.01); // tolerance
    let mesh = mesh_with_topology.to_polygon();
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(&mesh, &mut obj).unwrap();
}

/// Export the exact torus B-rep to STEP.
pub fn save_torus_step(shell: &Shell, path: &str) {
    let compressed = shell.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(path, display.to_string()).unwrap();
}
```

Steps inside `torus()`:

1. Place a vertex at `(0, 0, 1)`.
2. `rsweep` that vertex around +X (with `Rad(7.0)` > `2π`) to form a circular wire.
3. `rsweep` the wire around +Y to form the torus shell.

Tip: an angle ≥ `2π` closes the rotation; `Rad(7.0)` (~401°) ensures closure.

## Expose the torus module through lib.rs

**src/lib.rs**

```rust
pub mod shapes;
pub use shapes::{cube, save_obj, save_step, torus, save_torus_obj, save_torus_step};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub mod torus;

pub use cube::{cube, save_obj, save_step};
pub use torus::{torus, save_torus_obj, save_torus_step};
```

## Main entry point (bin)

`src/bin/torus.rs` stays tiny and calls into the library:

```rust
fn main() {
    let torus = truck_brep::torus();
    truck_brep::save_torus_obj(&torus, "output/torus.obj");
    truck_brep::save_torus_step(&torus, "output/torus.step");
}
```

<details>
<summary>Directory tree (for this section)</summary>

```
truck_brep/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs              # re-exports shapes
│  ├─ shapes/
│  │  ├─ mod.rs           # exports cube, torus, ...
│  │  ├─ cube.rs          # from previous section
│  │  └─ torus.rs         # torus() + save helpers
│  └─ bin/
│     ├─ cube.rs          # from previous section
│     └─ torus.rs         # calls into lib
└─ output/                # torus.obj, torus.step
```

</details>
