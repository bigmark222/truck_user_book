# Cylinder (rotational + planar + translational sweep)

Keep the geometry and exports inside the library crate (mirroring the cube and torus pages) so binaries stay tiny.

## Cylinder in `src/shapes/cylinder.rs`

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

/// Build a solid cylinder: rotational sweep → planar face → translational sweep.
pub fn cylinder() -> Solid {
    // Start with a vertex on the base circle
    let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, -1.0));

    // Spin the vertex around +Z to form a circular wire
    let circle: Wire = builder::rsweep(
        &vertex,
        Point3::new(0.0, 1.0, -1.0),
        Vector3::unit_z(),
        Rad(7.0), // > 2π ensures closure
    );

    // Close the wire into a planar disk
    let disk: Face =
        builder::try_attach_plane(&vec![circle]).expect("cannot attach plane");

    // Extrude the disk along +Z to make a solid
    builder::tsweep(&disk, 2.0 * Vector3::unit_z())
}

/// Triangulate and write an OBJ mesh.
pub fn save_cylinder_obj(solid: &Solid, path: &str) {
    let mesh_with_topology = solid.triangulation(0.01); // tolerance
    let mesh = mesh_with_topology.to_polygon();
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(&mesh, &mut obj).unwrap();
}

/// Export the exact cylinder B-rep to STEP.
pub fn save_cylinder_step(solid: &Solid, path: &str) {
    let compressed = solid.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(path, display.to_string()).unwrap();
}
```

Steps inside `cylinder()`:

1. Vertex at `(0, 0, -1)`.
2. `rsweep` around +Z to form a circular wire (base).
3. `try_attach_plane` closes the wire into a planar disk (face). Returns an error if the wire isn’t closed/planar.
4. `tsweep` the disk 2 units along +Z to create the solid cylinder.

## Expose the cylinder module through lib.rs

**src/lib.rs**

```rust
pub mod shapes;
pub use shapes::{
    cube, save_obj, save_step, torus, save_torus_obj, save_torus_step,
    cylinder, save_cylinder_obj, save_cylinder_step,
};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub mod torus;
pub mod cylinder;

pub use cube::{cube, save_obj, save_step};
pub use torus::{torus, save_torus_obj, save_torus_step};
pub use cylinder::{cylinder, save_cylinder_obj, save_cylinder_step};
```

## Main entry point (bin)

`src/bin/cylinder.rs` stays tiny and calls into the library:

```rust
fn main() {
    let cylinder = truck_brep::cylinder();
    truck_brep::save_cylinder_obj(&cylinder, "output/cylinder.obj");
    truck_brep::save_cylinder_step(&cylinder, "output/cylinder.step");
}
```

## Concepts recap

- `rsweep`: rotational sweep to build circular edges
- `try_attach_plane`: turn a closed planar wire into a face
- `tsweep`: translational sweep to create solids from faces

<details>
<summary>Directory tree (for this section)</summary>

```
truck_brep/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs              # re-exports shapes
│  ├─ shapes/
│  │  ├─ mod.rs           # exports cube, torus, cylinder, ...
│  │  ├─ cube.rs          # from 3.1
│  │  ├─ torus.rs         # from 3.2
│  │  └─ cylinder.rs      # this section
│  └─ bin/
│     ├─ cube.rs
│     ├─ torus.rs
│     └─ cylinder.rs      # calls into lib
└─ output/                # cylinder.obj, cylinder.step
```

</details>
