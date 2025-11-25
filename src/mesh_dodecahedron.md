# Dodecahedron

Add a dodecahedron to the `truck_meshes` library so the icosahedron can import it, with its own file.

![Cube in dodecahedron illustration](images/Cube_in_dodecahedron.jpg)

## Add the dodecahedron module

`src/lib.rs` additions:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

// ...keep earlier functions through octahedron...
pub mod dodecahedron;
pub use dodecahedron::dodecahedron;
```

## Construct Main Function

`src/dodecahedron.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

/// Dodecahedron built from a cube plus roof vertices.
pub fn dodecahedron() -> PolygonMesh {

    //PLACE STEP 1-4 HERE

}
```

#### Step 1: Define helper scalars and vertex positions
```rust
    let a = f64::sqrt(3.0) / 3.0; // half cube edge
    let l = 2.0 * a / (1.0 + f64::sqrt(5.0)); // half dodeca edge
    let d = f64::sqrt(1.0 - l * l); // other coordinate by Pythagoras

    let positions = vec![
        Point3::new(-a, -a, -a),
        Point3::new(a, -a, -a),
        Point3::new(a, a, -a),
        Point3::new(-a, a, -a),
        Point3::new(-a, -a, a),
        Point3::new(a, -a, a),
        Point3::new(a, a, a),
        Point3::new(-a, a, a),
        Point3::new(d, -l, 0.0),
        Point3::new(d, l, 0.0),
        Point3::new(-d, l, 0.0),
        Point3::new(-d, -l, 0.0),
        Point3::new(0.0, d, -l),
        Point3::new(0.0, d, l),
        Point3::new(0.0, -d, l),
        Point3::new(0.0, -d, -l),
        Point3::new(-l, 0.0, d),
        Point3::new(l, 0.0, d),
        Point3::new(l, 0.0, -d),
        Point3::new(-l, 0.0, -d),
    ];
```
<details>
<summary>Explanation</summary>

The hexahedron’s edges act as diagonals of regular pentagons, so each dodecahedron edge equals the cube edge divided by the golden ratio. The “roof” vertices have one coordinate at zero; solving the remaining coordinate with the Pythagorean theorem gives the helper scalars `a`, `l`, and `d` used for all 20 vertex positions.

</details>

#### Step 2: Build attribute set

```rust
    let attrs = StandardAttributes {
        positions,
        ..Default::default()
    };
```

#### Step 3: Define mesh faces
```rust
    let faces = Faces::from_iter([
        [4, 14, 5, 17, 16],
        [6, 13, 7, 16, 17],
        [6, 17, 5, 8, 9],
        [4, 16, 7, 10, 11],
        [4, 11, 0, 15, 14],
        [1, 8, 5, 14, 15],
        [6, 9, 2, 12, 13],
        [3, 10, 7, 13, 12],
        [1, 15, 0, 19, 18],
        [1, 18, 2, 9, 8],
        [3, 12, 2, 18, 19],
        [3, 19, 0, 11, 10],
    ]);
```
<details>
<summary>Explanation</summary>

Each line is one pentagonal face, listing which vertices to connect by their index in the `positions` list (e.g., `[4, 14, 5, 17, 16]` means positions 4→14→5→17→16). The vertices are ordered counter-clockwise as seen from outside the shape so the face normal points outward for correct lighting.

</details>

#### Step 4: Construct the mesh

```rust
    PolygonMesh::new(attrs, faces)
```

## Export the dodecahedron

Add `examples/dodecahedron.rs`:

```rust
fn main() {
    let mesh = truck_meshes::dodecahedron();
    truck_meshes::write_polygon_mesh(&mesh, "output/dodecahedron.obj");
}
```

Run it:

```bash
cargo run --example dodecahedron
```

<details>
<summary>File tree after this step</summary>

```
truck_meshes/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs
│  ├─ triangle.rs
│  ├─ square.rs
│  ├─ tetrahedron.rs
│  ├─ hexahedron.rs
│  ├─ octahedron.rs
│  └─ dodecahedron.rs
├─ examples/
│  ├─ triangle.rs
│  ├─ square.rs
│  ├─ tetrahedron.rs
│  ├─ hexahedron.rs
│  ├─ octahedron.rs
│  └─ dodecahedron.rs
└─ output/          # exported OBJ files (e.g., output/dodecahedron.obj)
```

</details>

<details>
<summary>Full code:</summary>

`src/lib.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

pub fn write_polygon_mesh(mesh: &PolygonMesh, path: &str) {
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(mesh, &mut obj).unwrap();
}

pub mod triangle;
pub use triangle::triangle;

pub mod square;
pub use square::square;

pub mod tetrahedron;
pub use tetrahedron::tetrahedron;

pub mod hexahedron;
pub use hexahedron::hexahedron;

pub mod octahedron;
pub use octahedron::octahedron;

pub mod dodecahedron;
pub use dodecahedron::dodecahedron;
```

`src/dodecahedron.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

pub fn dodecahedron() -> PolygonMesh {

    let a = f64::sqrt(3.0) / 3.0;
    let l = 2.0 * a / (1.0 + f64::sqrt(5.0));
    let d = f64::sqrt(1.0 - l * l);

    let positions = vec![
        Point3::new(-a, -a, -a),
        Point3::new(a, -a, -a),
        Point3::new(a, a, -a),
        Point3::new(-a, a, -a),
        Point3::new(-a, -a, a),
        Point3::new(a, -a, a),
        Point3::new(a, a, a),
        Point3::new(-a, a, a),
        Point3::new(d, -l, 0.0),
        Point3::new(d, l, 0.0),
        Point3::new(-d, l, 0.0),
        Point3::new(-d, -l, 0.0),
        Point3::new(0.0, d, -l),
        Point3::new(0.0, d, l),
        Point3::new(0.0, -d, l),
        Point3::new(0.0, -d, -l),
        Point3::new(-l, 0.0, d),
        Point3::new(l, 0.0, d),
        Point3::new(l, 0.0, -d),
        Point3::new(-l, 0.0, -d),
    ];

    let attrs = StandardAttributes {
        positions,
        ..Default::default()
    };

    let faces = Faces::from_iter([
        [4, 14, 5, 17, 16],
        [6, 13, 7, 16, 17],
        [6, 17, 5, 8, 9],
        [4, 16, 7, 10, 11],
        [4, 11, 0, 15, 14],
        [1, 8, 5, 14, 15],
        [6, 9, 2, 12, 13],
        [3, 10, 7, 13, 12],
        [1, 15, 0, 19, 18],
        [1, 18, 2, 9, 8],
        [3, 12, 2, 18, 19],
        [3, 19, 0, 11, 10],
    ]);

    PolygonMesh::new(attrs, faces)

}
```

`examples/dodecahedron.rs`:

```rust
fn main() {
    let mesh = truck_meshes::dodecahedron();
    truck_meshes::write_polygon_mesh(&mesh, "output/dodecahedron.obj");
}
```

</details>
