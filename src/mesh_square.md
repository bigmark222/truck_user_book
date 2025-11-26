# Square (Two Triangles)

Keep working in the same `truck_meshes` library so shapes can import each other, and place each shape in its own file.

Reference code: same setup as [triangle](mesh_first_triangle.md), but with 4 vertices and 2 faces.

## Add the square module

`src/lib.rs` additions:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

// ...keep `write_polygon_mesh` and `triangle()` above...
pub mod square; // add this
pub use square::square; // add this
```
## Construct Main Function

`src/square.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

/// A unit square in the XY plane made from two triangles.
pub fn square() -> PolygonMesh {

    //PLACE STEP 1-4 HERE

}
```

#### Step 1: Define vertex positions
```rust
    let positions = vec![
        Point3::new(0.0, 0.0, 0.0), // bottom-left
        Point3::new(1.0, 0.0, 0.0), // bottom-right
        Point3::new(1.0, 1.0, 0.0), // top-right
        Point3::new(0.0, 1.0, 0.0), // top-left
    ];
```
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
        [0, 1, 2], // bottom-right triangle
        [0, 2, 3], // top-left triangle
    ]);
```

Prefer a single quad? Swap the faces line in Step 3 for:

```rust
    let faces = Faces::from_iter([[0, 1, 2, 3]]);
```

#### Step 4: Construct the mesh
```rust
    PolygonMesh::new(attrs, faces)
```

## Export the square

Add `examples/square.rs`:

```rust
fn main() {
    let mesh = truck_meshes::square();
    truck_meshes::write_polygon_mesh(&mesh, "output/square.obj");
}
```

Run it:

```bash
cargo run --example square
```

<details>
<summary>File tree after this step</summary>

```
truck_meshes/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs
│  ├─ triangle.rs
│  └─ square.rs
├─ examples/
│  ├─ triangle.rs
│  └─ square.rs
└─ output/          # exported OBJ files (e.g., output/square.obj)
```

</details>

<details>
<summary>Full code:</summary>

`src/lib.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

/// Write any mesh to an OBJ file.
pub fn write_polygon_mesh(mesh: &PolygonMesh, path: &str) {
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(mesh, &mut obj).unwrap();
}

pub mod triangle;
pub use triangle::triangle;

pub mod square;
pub use square::square;
```

`src/square.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

pub fn square() -> PolygonMesh {
    let positions = vec![
        Point3::new(0.0, 0.0, 0.0),
        Point3::new(1.0, 0.0, 0.0),
        Point3::new(1.0, 1.0, 0.0),
        Point3::new(0.0, 1.0, 0.0),
    ];

    let attrs = StandardAttributes {
        positions,
        ..Default::default()
    };

    let faces = Faces::from_iter([
        [0, 1, 2],
        [0, 2, 3],
    ]);

    PolygonMesh::new(attrs, faces)
}
```

`examples/square.rs`:

```rust
fn main() {
    let mesh = truck_meshes::square();
    truck_meshes::write_polygon_mesh(&mesh, "output/square.obj");
}
```

</details>
