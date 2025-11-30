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


## Create file `src/shapes/torus.rs`:

```rust
use truck_modeling::prelude::*;
```

```rust
/// Build a torus shell with nested rotational sweeps.
pub fn torus() -> Shell {

  // STEPS 1-3 HERE

}
```

### 1. Place a vertex at `(0, 0, 1)`.

```rust
  let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, 1.0));
```

### 2. Spin the vertex around +X to form a circular wire.

```rust
  let circle: Wire = builder::rsweep(
      &vertex,
      Point3::new(0.0, 0.5, 1.0), // point on rotation axis
      Vector3::unit_x(),         // axis direction
      Rad(7.0),                  // > 2π ensures closure
  );
```

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

### 3. Spin the circle around +Y to form the torus shell.

```rust
  builder::rsweep(&circle, Point3::origin(), Vector3::unit_y(), Rad(7.0))
```

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

## Expose the torus module through lib.rs

**src/lib.rs**

```rust
pub mod helpers;
pub mod shapes;

pub use helpers::{save_obj, save_step_any};
pub use shapes::{cube, torus};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub mod torus;

pub use cube::cube;
pub use torus::torus;
```

## Example entry point

`examples/torus.rs` stays tiny and calls into the library:

```rust
fn main() {
    let torus = truck_brep::torus();
    truck_brep::save_obj(&torus, "output/torus.obj");
    truck_brep::save_step_any(&torus, "output/torus.step");
}
```

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
│  │  ├─ mod.rs           # exports cube, torus, ...
│  │  ├─ cube.rs          # from previous section
│  │  └─ torus.rs         # torus()
│  └─ examples/
│     ├─ cube.rs          # from previous section
│     └─ torus.rs         # calls into lib
└─ output/                # torus.obj, torus.step
```

</details>

<details>
<summary>Complete code (lib + shapes + example)</summary>

**src/lib.rs**

```rust
pub mod helpers;
pub mod shapes;

pub use helpers::{save_obj, save_step_any};
pub use shapes::{cube, torus};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub mod torus;

pub use cube::cube;
pub use torus::torus;
```

**src/shapes/torus.rs**

```rust
use truck_modeling::prelude::*;

pub fn torus() -> Shell {
    let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, 1.0));
    let circle: Wire = builder::rsweep(
        &vertex,
        Point3::new(0.0, 0.5, 1.0),
        Vector3::unit_x(),
        Rad(7.0),
    );
    builder::rsweep(&circle, Point3::origin(), Vector3::unit_y(), Rad(7.0))
}
```

**examples/torus.rs**

```rust
fn main() {
    let torus = truck_brep::torus();
    truck_brep::save_obj(&torus, "output/torus.obj");
    truck_brep::save_step_any(&torus, "output/torus.step");
}
```

</details>
