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


### `src/torus.rs`:

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};
```

```rust
/// Build a torus shell with nested rotational sweeps.
pub fn torus() -> Shell {

  // STEPS 1-3 HERE

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

Keep helpers in `src/lib.rs` and torus in `src/torus.rs`.

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
│  ├─ lib.rs              # helpers + re-exports
│  ├─ cube.rs             # from previous section (optional)
│  └─ torus.rs            # torus()
├─ examples/
│  ├─ cube.rs             # from previous section
│  └─ torus.rs            # calls into lib
└─ output/                # torus.obj, torus.step
```

</details>

<details>
<summary>Complete code (helpers in lib, torus module + example)</summary>

**src/lib.rs**

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

pub mod torus;
pub use torus::torus;

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

**src/torus.rs**

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
