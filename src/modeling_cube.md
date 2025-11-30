# Cube

Model a precise B-rep cube in Truck

<details>
  <summary><strong>B-rep building blocks</strong></summary>
  <ul>
    <li><b>Vertex:</b> point</li>
    <li><b>Edge:</b> curve between two vertices</li>
    <li><b>Wire:</b> sequence of edges forming a loop</li>
    <li><b>Face:</b> bounded by a closed wire</li>
    <li><b>Shell:</b> group of faces</li>
    <li><b>Solid:</b> closed shell forming a volume</li>
  </ul>
</details>

## Build a cube with `tsweep`

<details>
  <summary><strong>What <code>tsweep</code> does</strong></summary>

  <p><code>tsweep</code> takes a geometric element and pushes it in a straight line to create the next-level element:</p>

  <ul>
    <li>sweep a <strong>vertex</strong> → you get an <strong>edge</strong></li>
    <li>sweep an <strong>edge</strong> → you get a <strong>face</strong></li>
    <li>sweep a <strong>face</strong> → you get a <strong>solid</strong></li>
  </ul>

  <p>A cube is built by sweeping three times along the X, Y, and Z directions.</p>
</details>

### `src/cube.rs`

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};
```

```rust

/// Build a solid cube by chaining translational sweeps.
pub fn cube() -> Solid {

  // STEPS 1-4 HERE

}
```

#### 1. Place the first vertex at `(-1, 0, -1)`.

```rust
  let vertex: Vertex = builder::vertex(Point3::new(-1.0, 0.0, -1.0));
```
<details>
<summary>what this code does</summary>

`builder::vertex` lifts a raw point into a B-rep `Vertex`, giving us a manipulable geometric anchor for the sweeps that follow.
</details>
<details>
<summary>visual</summary>

```
            y
            │
            │
(x=-1)  ●   │
 start    \ │
  here     \│
            └──── x
           /
          z

Point = (-1, 0, -1)
```

</details>

#### 2. Sweep 2 units along +Z → edge.

```rust
  let edge:   Edge   = builder::tsweep(&vertex, 2.0 * Vector3::unit_z());
```
<details>
<summary>what this code does</summary>

`builder::tsweep` clones the vertex and moves the copy by `2.0 * +Z`—since `+Z` is the unit vector `(0, 0, 1)`, the translation is `(0, 0, 2)` (two units up the Z axis).  
It then returns the **Edge** spanning between the original point and the shifted one.

</details>

<details>
<summary>visual</summary>

```
Before sweep:

   ●  (vertex)

After sweeping along +Z:

   ●──────────●
    (start)   (end)

This is the new Edge.
```

</details>

#### 3. Sweep edge 2 units along +X → rectangular face.

```rust
  let face:   Face   = builder::tsweep(&edge,   2.0 * Vector3::unit_x());
```
<details>
<summary>what this code does</summary>

The edge is duplicated and shifted +X by 2.0; `tsweep` stitches the original and shifted edges into a ruled surface, yielding a rectangular `Face`.
</details>

<details>
<summary>visual</summary>

```
Sweep direction +X:

   ●──────────●
   |          |  
   |          |
   |          |
   │          │ 
   ●──────────●

That forms a rectangular Face.
```

</details>

#### 4. Sweep face 2 units along +Y → solid cube.

```rust
  builder::tsweep(&face, 2.0 * Vector3::unit_y())
```
<details>
<summary>what this code does</summary>

Sweeps the rectangular face upward by 2.0 along +Y, then caps the start and end to form a closed shell; because the face is planar and bounded, the result is a watertight solid cube.
</details>

<details>
<summary>visual</summary>

```
Sweeping the rectangular face upward (+Y) forms the cube volume:

               ●─────────●
              /|        /|
             / |       / |
            ●─────────●  |
            │  |      │  |
            │  ●──────┼──●
            │ /       │ /
            ●─────────●
                (full solid)

Direction of final sweep: +Y

```

</details>

## Update `src/lib.rs` to expose `cube`

```rust
pub mod cube;
pub use cube::cube;
```

## Example entry point

Use the shared export helpers from `src/lib.rs`, and the `cube` function we just created.

`examples/cube.rs` just calls the library functions:

```rust
fn main() {
    let cube = truck_brep::cube();
    truck_brep::save_obj(&cube, "output/cube.obj");
    truck_brep::save_step_any(&cube, "output/cube.step");
}
```

This keeps the binary minimal and lets other sections reuse the same helpers.

<details>
<summary>Directory tree</summary>

```
truck_brep/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs              # save_obj, save_step_any, re-exports shapes
│  └─ cube.rs             # cube()
├─ examples/
│  └─ cube.rs             # calls into lib
└─ output/                # cube.obj, cube.step
```

</details>

</details>

<details>
<summary>Complete code (helpers in lib, cube module + example)</summary>

**src/lib.rs**

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

pub mod cube;
pub use cube::cube;

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

**src/cube.rs**

```rust
use truck_modeling::prelude::*;

pub fn cube() -> Solid {
    let vertex: Vertex = builder::vertex(Point3::new(-1.0, 0.0, -1.0));
    let edge: Edge = builder::tsweep(&vertex, 2.0 * Vector3::unit_z());
    let face: Face = builder::tsweep(&edge, 2.0 * Vector3::unit_x());
    builder::tsweep(&face, 2.0 * Vector3::unit_y())
}
```

**examples/cube.rs**

```rust
fn main() {
    let cube = truck_brep::cube();
    truck_brep::save_obj(&cube, "output/cube.obj");
    truck_brep::save_step_any(&cube, "output/cube.step");
}
```

</details>
