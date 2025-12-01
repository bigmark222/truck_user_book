# Torus

Build a torus (donut) with rotational sweeps, keeping the geometry and exports in the library just like the cube section. For the cylinder example, see `modeling_cylinder.md`.

## Build a torus with `rsweep`

<details>
  <summary><strong>What <code>rsweep</code> does</strong></summary>

  <p><code>rsweep</code> spins a geometric element around an axis to create the next-level element:</p>

  <ul>
    <li>spin a <strong>vertex</strong> → you get a <strong>wire (circle)</strong></li>
    <li>spin a <strong>wire</strong> → you get a <strong>shell</strong></li>
  </ul>

  <p>A torus is built by two rotations: one to form the circle, another to spin that circle into the donut.</p>
</details>


### `src/torus.rs`

```rust
use truck_modeling::*;
```

```rust
pub fn torus() -> Solid {
  // STEPS 1-3 GO HERE

}
```

#### 1. Place a vertex at `(0, 0, 1)`.

```rust
  let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, 1.0));
```
<details>
<summary>what this code does</summary>

Creates a B-rep vertex at (0, 0, 1) to act as the seed point for the rotational sweeps.
</details>

<details>
<summary>visual</summary>

```
        y
        │
        │
   ●    │   (start at z = 1)
        │
        └──── x
       /
      z
```

</details>

#### 2. Spin the vertex around +X to form a circular wire.

```rust
  let circle: Wire = builder::rsweep(
      &vertex,
      Point3::new(0.0, 0.5, 1.0), // point on rotation axis
      Vector3::unit_x(),         // axis direction
      Rad(7.0),                  // > 2π ensures closure
  );
```
<details>
<summary>what this code does</summary>

`rsweep` clones the vertex and spins it around the +X axis through (0, 0.5, 1.0); the start and end positions connect into a circular wire.
</details>

<details>
<summary>visual</summary>

```
Axis: +X through (0, 0.5, 1.0)

   (spins around X)
       ^
       |
   ●---+---●  (wire)
       |
     axis
```

</details>

#### 3. Spin the circle around +Y to form the torus shell.

```rust
  builder::rsweep(&circle, Point3::origin(), Vector3::unit_y(), Rad(7.0))
```
<details>
<summary>what this code does</summary>

Rotates the circle about the Y axis at the origin; the swept surface forms the torus shell.
</details>

<details>
<summary>visual</summary>

```
Second rotation around +Y wraps the circle into a torus:

      ●─────●
     /       \
    ●         ●
     \       /
      ●─────●

 ```

</details>

## Update `src/lib.rs` to expose `torus`

```rust
pub mod torus;
pub use torus::torus;
```

## Example entry point

`examples/torus.rs` stays tiny and calls into the library:

```rust
fn main() {
    let torus = truck_brep::torus();
    truck_brep::save_obj(&torus, "output/torus.obj").unwrap();
    truck_brep::save_step(&torus, "output/torus.step").unwrap();
}
```

<br>

<details>
<summary>Directory tree (for this section)</summary>

```
truck_brep/
├─ src/
│  ├─ lib.rs      # helpers + re-exports
│  ├─ cube.rs     # from previous section (optional)
│  └─ torus.rs    # torus()
├─ examples/
│  ├─ cube.rs
│  └─ torus.rs
└─ output/        # created at runtime
```

</details>

<details>
<summary>Complete code</summary>

**src/lib.rs**

```rust
use std::{fs, io, path::Path};

use truck_meshalgo::prelude::*;
use truck_modeling::*;
use truck_stepio::out::{CompleteStepDisplay, StepModel};
use truck_topology::compress::{CompressedShell, CompressedSolid};

pub mod cube;
pub use cube::cube;

pub mod torus;
pub use torus::torus;

/// Helper to compress modeling shapes into STEP-compatible data.
pub trait StepCompress {
    type Compressed;
    fn compress_for_step(&self) -> Self::Compressed;
}

impl StepCompress for Shell {
    type Compressed = CompressedShell<Point3, Curve, Surface>;
    fn compress_for_step(&self) -> Self::Compressed { self.compress() }
}

impl StepCompress for Solid {
    type Compressed = CompressedSolid<Point3, Curve, Surface>;
    fn compress_for_step(&self) -> Self::Compressed { self.compress() }
}

/// Export any B-rep (Solid or Shell) to STEP.
pub fn save_step<T, P>(brep: &T, path: P) -> io::Result<()>
where
    T: StepCompress,
    for<'a> StepModel<'a, Point3, Curve, Surface>: From<&'a T::Compressed>,
    P: AsRef<Path>,
{
    let path = path.as_ref();
    if let Some(parent) = path.parent() {
        fs::create_dir_all(parent)?;
    }
    let compressed = brep.compress_for_step();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    fs::write(path, display.to_string())
}

/// Triangulate any B-rep (Solid or Shell) and write an OBJ mesh.
pub fn save_obj(shape: &impl MeshableShape, path: impl AsRef<Path>) -> io::Result<()> {
    let mesh = shape.triangulation(0.01).to_polygon();
    let path = path.as_ref();
    if let Some(parent) = path.parent() {
        fs::create_dir_all(parent)?;
    }
    let mut obj = fs::File::create(path)?;
    obj::write(&mesh, &mut obj).map_err(|err| io::Error::new(io::ErrorKind::Other, err))
}

```

**src/torus.rs**

```rust
use truck_modeling::*;

pub fn torus() -> Shell {
    let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, 1.0));
    let circle: Wire = builder::rsweep(
        &vertex,
        Point3::new(0.0, 0.5, 1.0), // point on rotation axis
        Vector3::unit_x(),         // axis direction
        Rad(7.0),                  // > 2π ensures closure
    );
    builder::rsweep(&circle, Point3::origin(), Vector3::unit_y(), Rad(7.0))
}
```

**examples/torus.rs**

```rust
fn main() {
    let torus = truck_brep::torus();
    truck_brep::save_obj(&torus, "output/torus.obj").unwrap();
    truck_brep::save_step_any(&torus, "output/torus.step").unwrap();
}
```

</details>
