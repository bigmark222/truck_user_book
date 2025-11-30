# Cylinder (rotational + planar + translational sweep)

Keep the geometry and exports inside the library crate (mirroring the cube and torus pages) so binaries stay tiny.

## Build a cylinder with `rsweep`, `try_attach_plane`, and `tsweep`

<details>
  <summary><strong>What happens at each step</strong></summary>

  <ul>
    <li><strong>Rotational sweep (`rsweep`):</strong> spin a vertex to make a circular wire.</li>
    <li><strong>Plane attachment:</strong> cap the wire into a disk with <code>try_attach_plane</code>.</li>
    <li><strong>Translational sweep (`tsweep`):</strong> extrude the disk to form the solid.</li>
  </ul>
</details>

### `src/cylinder.rs`:

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};
```

```rust
/// Build a solid cylinder: rotational sweep → planar face → translational sweep.
pub fn cylinder() -> Solid {

  // STEPS 1-4 HERE

}
```

#### 1. Place a vertex on the base circle at `(0, 0, -1)`.

```rust
  let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, -1.0));
```

<details>
<summary>visual</summary>

```
          y
          │
          │
   ●      │   (start at z = -1)
          │
          └──── x
         /
        z
```

</details>

#### 2. Spin the vertex around +Z to form a circular wire.

```rust
  let circle: Wire = builder::rsweep(
      &vertex,
      Point3::new(0.0, 1.0, -1.0), // point on rotation axis
      Vector3::unit_z(),          // axis direction
      Rad(7.0),                   // > 2π ensures closure
  );
```

<details>
<summary>visual</summary>

```
Axis: +Z through (0, 1, -1)

     ●---●---●   (circle in XY plane at z = -1)
      \  |  /
       \ | /
         +
        axis
```

</details>

#### 3. Cap the wire into a planar disk.

```rust
  let disk: Face =
      builder::try_attach_plane(&vec![circle]).expect("cannot attach plane");
```

<details>
<summary>visual</summary>

```
The circular wire is filled to a disk (face) if the wire is closed and planar.

     +------+
    /        \
   |    ●     |
    \        /
     +------+
```

</details>

#### 4. Extrude the disk 2 units along +Z to make the solid.

```rust
  builder::tsweep(&disk, 2.0 * Vector3::unit_z())
```

<details>
<summary>visual</summary>

```
Sweeping the disk upward (+Z) forms the cylinder volume:

     ●───────●
    /|       /|
   ●───────●  |
   | |      | |
   | ●──────|-●
   |/       |/
   ●───────●
```

</details>

## Update `src/lib.rs` to expose `cylinder`

```rust
pub mod cylinder;
pub use cylinder::cylinder;
```

Keep helpers in lib.rs and cylinder in `src/cylinder.rs`.

## Example entry point

`examples/cylinder.rs` stays tiny and calls into the library:

```rust
fn main() {
    let cylinder = truck_brep::cylinder();
    truck_brep::save_obj(&cylinder, "output/cylinder.obj");
    truck_brep::save_step_any(&cylinder, "output/cylinder.step");
}
```

<br>

<details>
<summary>Directory tree (for this section)</summary>

```
truck_brep/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs              # helpers + re-exports
│  ├─ cube.rs
│  ├─ torus.rs
│  └─ cylinder.rs         # this section
├─ examples/
│  ├─ cube.rs
│  ├─ torus.rs
│  └─ cylinder.rs         # calls into lib
└─ output/                # cylinder.obj, cylinder.step
```

</details>

<details>
<summary>Complete code (helpers in lib, cylinder module + example)</summary>

**src/lib.rs**

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

pub mod cube;
pub use cube::cube;

pub mod torus;
pub use torus::torus;

pub mod cylinder;
pub use cylinder::cylinder;

pub fn cylinder() -> Solid {
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

/// Export any B-rep (Solid or Shell) to STEP.
pub fn save_step_any<T, P>(brep: &T, path: P) -> std::io::Result<()>
where
    T: Compress,
    for<'a> StepModel<'a>: From<&'a T>,
    P: AsRef<std::path::Path>,
{
    let compressed = brep.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(path, display.to_string())
}

/// Triangulate any B-rep (Solid or Shell) and write an OBJ mesh.
pub fn save_obj(shape: &impl MeshedShape, path: &str) {
    let mesh = shape.triangulation(0.01).to_polygon();
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(&mesh, &mut obj).unwrap();
}
```

**src/cylinder.rs**

```rust
use truck_modeling::prelude::*;

pub fn cylinder() -> Solid {
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
```

**examples/cylinder.rs**

```rust
fn main() {
    let cylinder = truck_brep::cylinder();
    truck_brep::save_obj(&cylinder, "output/cylinder.obj");
    truck_brep::save_step_any(&cylinder, "output/cylinder.step");
}
```

</details>
