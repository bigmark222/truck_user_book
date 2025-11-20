# Cube

Model a precise B-rep cube with Truck—exact geometry built from vertices, edges, faces, and a solid.

Reference: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter3/src/section3_1.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter3/src/section3_1.rs)

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

## Build a cube with `tsweep`

`tsweep` sweeps a geometric element along a vector to produce the next higher element. Four sweeps make a cube:

```rust
use truck_modeling::prelude::*;

fn cube() -> Solid {
    let vertex: Vertex = builder::vertex(Point3::new(-1.0, 0.0, -1.0));
    let edge:   Edge   = builder::tsweep(&vertex, 2.0 * Vector3::unit_z());
    let face:   Face   = builder::tsweep(&edge,   2.0 * Vector3::unit_x());
    builder::tsweep(&face, 2.0 * Vector3::unit_y())
}
```

Steps:

1. Place the first vertex at `(-1, 0, -1)`.
2. Sweep 2 units along +Z → edge.
3. Sweep edge 2 units along +X → rectangular face.
4. Sweep face 2 units along +Y → solid cube.

## Output the geometry

### Option 1: mesh + OBJ (for viewers/renderers)

```rust
use truck_meshalgo::prelude::*;

fn save_obj(solid: &Solid, path: &str) {
    // Triangulate with 0.01 tolerance, then merge faces
    let mesh_with_topology = solid.triangulation(0.01);
    let mesh = mesh_with_topology.to_polygon();

    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(&mesh, &mut obj).unwrap();
}
```

- Tolerance `0.01` controls mesh approximation.
- Works in Blender, ParaView, etc.

### Option 2: exact STEP export (for CAD)

```rust
use truck_stepio::{CompleteStepDisplay, StepModel};

fn save_step(solid: &Solid, path: &str) {
    let compressed = solid.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    let step_string = display.to_string();
    std::fs::write(path, &step_string).unwrap();
}
```

Open the STEP in FreeCAD, SolidWorks, CATIA, NX, Fusion 360, Onshape, etc.

## Main entry point

```rust
fn main() {
    let cube = cube();
    save_obj(&cube, "cube.obj");
    save_step(&cube, "cube.step");
}
```

This completes 3.1 Cube.
