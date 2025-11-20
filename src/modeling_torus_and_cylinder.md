# Torus and Cylinder

Create two classic curved solids with Truck’s B-rep tools: a torus (donut) and a cylinder. Both rely on sweeping operations.

Reference: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter3/src/section3_2.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter3/src/section3_2.rs)

## Torus (rotational sweep)

Use `rsweep` to rotate geometry around an axis. Three lines build the torus surface (a `Shell`):

```rust
use truck_modeling::prelude::*;

fn torus() -> Shell {
    let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, 1.0));
    let circle: Wire = builder::rsweep(
        &vertex,
        Point3::new(0.0, 0.5, 1.0), // point on rotation axis
        Vector3::unit_x(),         // axis direction
        Rad(7.0),                  // angle (~401°) to ensure closure
    );
    builder::rsweep(&circle, Point3::origin(), Vector3::unit_y(), Rad(7.0))
}
```

Steps:

1. Start with a vertex at `(0, 0, 1)`.
2. `rsweep` the vertex around +X to form a circular wire.
3. `rsweep` the wire around +Y to form the torus shell.

Tip: an angle ≥ `2π` closes the rotation; `Rad(7.0)` (~401°) ensures closure.

## Cylinder (rotational + planar + translational sweep)

Combine `rsweep`, `try_attach_plane`, and `tsweep` to make a solid cylinder:

```rust
use truck_modeling::prelude::*;

fn cylinder() -> Solid {
    let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, -1.0));

    let circle: Wire = builder::rsweep(
        &vertex,
        Point3::new(0.0, 1.0, -1.0),
        Vector3::unit_z(),
        Rad(7.0),
    );

    let disk: Face =
        builder::try_attach_plane(&vec![circle]).expect("cannot attach plane");

    builder::tsweep(&disk, 2.0 * Vector3::unit_z())
}
```

Steps:

1. Vertex at `(0, 0, -1)`.
2. `rsweep` around +Z to form a circular wire (base).
3. `try_attach_plane` closes the wire into a planar disk (face). Returns an error if the wire isn’t closed/planar.
4. `tsweep` the disk 2 units along +Z to create the solid cylinder.

## Saving shapes (OBJ + STEP)

```rust
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

fn save_shape(solid: &Solid, name: &str) {
    // Mesh + OBJ
    let mesh_with_topology = solid.triangulation(0.01); // tolerance
    let mesh = mesh_with_topology.to_polygon();
    let mut obj = std::fs::File::create(format!("{name}.obj")).unwrap();
    obj::write(&mesh, &mut obj).unwrap();

    // STEP (exact B-rep)
    let compressed = solid.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(format!("{name}.step"), display.to_string()).unwrap();
}
```

## Main entry point

```rust
fn main() {
    save_shape(&torus(), "torus");
    save_shape(&cylinder(), "cylinder");
}
```

## Concepts recap

- `rsweep`: rotational sweep for circular shapes (torus loops, cylinder bases)
- `try_attach_plane`: turn a closed planar wire into a filled face
- `tsweep`: translational sweep to create solids from faces
