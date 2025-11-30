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

## Create file `src/shapes/cylinder.rs`:

```rust
use truck_modeling::prelude::*;
```

```rust
/// Build a solid cylinder: rotational sweep → planar face → translational sweep.
pub fn cylinder() -> Solid {

  // STEPS 1-4 HERE

}
```

### 1. Place a vertex on the base circle at `(0, 0, -1)`.

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

### 2. Spin the vertex around +Z to form a circular wire.

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

### 3. Cap the wire into a planar disk.

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

### 4. Extrude the disk 2 units along +Z to make the solid.

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

## Expose the cylinder module through lib.rs

**src/lib.rs**

```rust
pub mod helpers;
pub mod shapes;

pub use helpers::{save_obj, save_step_any};
pub use shapes::{cube, cylinder, torus};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub mod torus;
pub mod cylinder;

pub use cube::cube;
pub use torus::torus;
pub use cylinder::cylinder;
```

## Example entry point

`examples/cylinder.rs` stays tiny and calls into the library:

```rust
fn main() {
    let cylinder = truck_brep::cylinder();
    truck_brep::save_obj(&cylinder, "output/cylinder.obj");
    truck_brep::save_step_any(&cylinder, "output/cylinder.step");
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
│  ├─ lib.rs              # re-exports helpers + shapes
│  ├─ helpers/
│  │  └─ mod.rs           # shared OBJ/STEP helpers
│  ├─ shapes/
│  │  ├─ mod.rs           # exports cube, torus, cylinder, ...
│  │  ├─ cube.rs          # from 3.1
│  │  ├─ torus.rs         # from 3.2
│  │  └─ cylinder.rs      # this section
│  └─ examples/
│     ├─ cube.rs
│     ├─ torus.rs
│     └─ cylinder.rs      # calls into lib
└─ output/                # cylinder.obj, cylinder.step
```

</details>

<details>
<summary>Complete code (lib + shapes + example)</summary>

**src/lib.rs**

```rust
pub mod helpers;
pub mod shapes;

pub use helpers::{save_obj, save_step_any};
pub use shapes::{cube, cylinder, torus};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub mod torus;
pub mod cylinder;

pub use cube::cube;
pub use torus::torus;
pub use cylinder::cylinder;
```

**src/shapes/cylinder.rs**

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
