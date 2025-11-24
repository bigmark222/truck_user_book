# Torus

Build a torus (donut) with rotational sweeps. For the cylinder example, see `modeling_cylinder.md`.

Reference: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter3/src/section3_2.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter3/src/section3_2.rs)

## Torus (rotational sweep)

Use `rsweep` twice: first to spin a vertex into a circle, then to spin that circle into the torus shell.

```rust
use truck_modeling::prelude::*;

fn torus() -> Shell {
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
```

Tip: an angle ≥ `2π` closes the rotation; `Rad(7.0)` (~401°) ensures closure.

## Save the torus (OBJ + STEP)

```rust
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

fn save_shape(shell: &Shell, name: &str) {
    // Mesh + OBJ
    let mesh_with_topology = shell.triangulation(0.01); // tolerance
    let mesh = mesh_with_topology.to_polygon();
    let mut obj = std::fs::File::create(format!("{name}.obj")).unwrap();
    obj::write(&mesh, &mut obj).unwrap();

    // STEP (exact B-rep)
    let compressed = shell.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(format!("{name}.step"), display.to_string()).unwrap();
}

fn main() {
    save_shape(&torus(), "torus");
}
```

## Concepts recap

- `rsweep`: rotational sweep for circular shapes
- `Shell`: collection of faces without interior volume (torus surface)
