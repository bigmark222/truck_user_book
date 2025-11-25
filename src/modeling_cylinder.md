# Cylinder (rotational + planar + translational sweep)

```rust
use truck_modeling::prelude::*;

fn cylinder() -> Solid {
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
```

Steps:

1. Vertex at `(0, 0, -1)`.
2. `rsweep` around +Z to form a circular wire (base).
3. `try_attach_plane` closes the wire into a planar disk (face). Returns an error if the wire isn’t closed/planar.
4. `tsweep` the disk 2 units along +Z to create the solid cylinder.

## Save the cylinder (OBJ + STEP)

```rust
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

fn save_shape(solid: &Solid, name: &str) {
    // Mesh + OBJ
    let mesh_with_topology = solid.triangulation(0.01); // tolerance
    let mesh = mesh_with_topology.to_polygon();
    let mut obj = std::fs::File::create(format!("output/{name}.obj")).unwrap();
    obj::write(&mesh, &mut obj).unwrap();

    // STEP (exact B-rep)
    let compressed = solid.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(format!("output/{name}.step"), display.to_string()).unwrap();
}

fn main() {
    save_shape(&cylinder(), "cylinder");
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

<details>
<summary>Complete code (src/bin/cylinder.rs)</summary>

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

fn cylinder() -> Solid {
    let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, -1.0));
    let circle: Wire = builder::rsweep(
        &vertex,
        Point3::new(0.0, 1.0, -1.0),
        Vector3::unit_z(),
        Rad(7.0),
    );
    let disk: Face = builder::try_attach_plane(&vec![circle]).expect("cannot attach plane");
    builder::tsweep(&disk, 2.0 * Vector3::unit_z())
}

fn save_shape(solid: &Solid, name: &str) {
    let mesh_with_topology = solid.triangulation(0.01);
    let mesh = mesh_with_topology.to_polygon();
    let mut obj = std::fs::File::create(format!("output/{name}.obj")).unwrap();
    obj::write(&mesh, &mut obj).unwrap();

    let compressed = solid.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(format!("output/{name}.step"), display.to_string()).unwrap();
}

fn main() {
    save_shape(&cylinder(), "cylinder");
}
```

</details>
