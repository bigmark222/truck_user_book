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



## Create file `src/shapes/cube.rs`:

```rust
use truck_modeling::prelude::*;
```

```rust

/// Build a solid cube by chaining translational sweeps.
pub fn cube() -> Solid {

  // STEPS 1-4 HERE

}
```

### 1. Place the first vertex at `(-1, 0, -1)`.

```rust
  let vertex: Vertex = builder::vertex(Point3::new(-1.0, 0.0, -1.0));
```
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

### 2. Sweep 2 units along +Z → edge.

```rust
  let edge:   Edge   = builder::tsweep(&vertex, 2.0 * Vector3::unit_z());
```

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

### 3. Sweep edge 2 units along +X → rectangular face.

```rust
  let face:   Face   = builder::tsweep(&edge,   2.0 * Vector3::unit_x());
```

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

### 4. Sweep face 2 units along +Y → solid cube.

```rust
  builder::tsweep(&face, 2.0 * Vector3::unit_y())
```

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


## Expose the cube module through lib.rs

**src/lib.rs**

```rust
pub mod helpers;
pub mod shapes;

pub use helpers::{save_obj, save_step_any};
pub use shapes::cube;
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub use cube::cube;
```


## Example entry point

Use the shared export helpers from `src/helpers/mod.rs`, and the `cube` function we just created.

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
│  ├─ lib.rs              # re-exports helpers + shapes
│  ├─ helpers/
│  │  └─ mod.rs           # save_obj, save_step_any
│  ├─ shapes/
│  │  ├─ mod.rs
│  │  └─ cube.rs          # cube()
│  └─ examples/
│     └─ cube.rs          # calls into lib
└─ output/                # cube.obj, cube.step
```

</details>

</details>

<details>
<summary>Complete code (lib, shapes, example)</summary>

**src/shapes/cube.rs**

```rust
use truck_modeling::prelude::*;

pub fn cube() -> Solid {
    let vertex: Vertex = builder::vertex(Point3::new(-1.0, 0.0, -1.0));
    let edge: Edge = builder::tsweep(&vertex, 2.0 * Vector3::unit_z());
    let face: Face = builder::tsweep(&edge, 2.0 * Vector3::unit_x());
    builder::tsweep(&face, 2.0 * Vector3::unit_y())
}
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub use cube::cube;
```

**src/lib.rs**

```rust
pub mod helpers;
pub mod shapes;

pub use helpers::{save_obj, save_step_any};
pub use shapes::cube;
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




