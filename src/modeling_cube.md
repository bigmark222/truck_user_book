# Cube

Model a precise B-rep cube in Truck

<details>
  <summary><strong>B-rep Building Blocks</strong></summary>

  <ul>
    <li><b>Vertex</b> → a point in 3D space.</li>
    <li><b>Edge</b> → a curve connecting two vertices (line, arc, spline, etc.).</li>
    <li><b>Wire</b> → an ordered loop of edges; closed wires bound faces.</li>
    <li><b>Face</b> → a surface patch bounded by one outer wire (plus optional inner wires for holes).</li>
    <li><b>Shell</b> → a connected set of stitched faces.</li>
    <li><b>Solid</b> → a closed, watertight shell enclosing a volume.</li>
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
use truck_modeling::*;
```

```rust
pub fn cube() -> Solid {
  // STEPS 1-4 GO HERE

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

`examples/cube.rs` calls into the library and uses the helper wrappers that create output folders automatically:

```rust
fn main() {
    let cube = truck_brep::cube();
    truck_brep::save_obj(&cube, "output/cube.obj").unwrap();
    truck_brep::save_step(&cube, "output/cube.step").unwrap();
}
```

This keeps the binary minimal and lets other sections reuse the same helpers.

<br>

<details>
<summary>Directory tree</summary>

```
truck_brep/
├─ src/
│  ├─ lib.rs      # helpers + re-exports
│  └─ cube.rs     # cube()
├─ examples/
│  └─ cube.rs     # calls into lib
└─ output/        # generated OBJ/STEP (created at runtime)
```

</details>

</details>

<details>
<summary>Complete code</summary>

**src/lib.rs** (imports/re-exports abbreviated)

```rust
use std::{fs, io, path::Path};

use truck_meshalgo::prelude::*;
use truck_modeling::*;
use truck_stepio::out::{CompleteStepDisplay, StepModel};
use truck_topology::compress::{CompressedShell, CompressedSolid};

pub mod cube;
pub use cube::cube;
// torus, cylinder, bottle modules are exported similarly

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

pub fn save_step_any<T, P>(brep: &T, path: P) -> io::Result<()>
where
    T: StepCompress,
    for<'a> StepModel<'a, Point3, Curve, Surface>: From<&'a T::Compressed>,
    P: AsRef<Path>,
{
    // ...
}

pub fn save_step<T, P>(brep: &T, path: P) -> io::Result<()> { save_step_any(brep, path) }

pub fn save_obj(shape: &impl MeshableShape, path: impl AsRef<Path>) -> io::Result<()> {
    // ...
}
```

**src/cube.rs**

```rust
use truck_modeling::*;

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
    truck_brep::save_obj(&cube, "output/cube.obj").unwrap();
    truck_brep::save_step(&cube, "output/cube.step").unwrap();
}
```

</details>
