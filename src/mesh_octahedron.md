# Octahedron

## Add the octahedron module

`src/lib.rs` additions:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

// ...keep earlier functions: write_polygon_mesh, triangle, square, tetrahedron, hexahedron...
pub mod octahedron; // add this
pub use octahedron::octahedron; // add this
```

## Construct Main Function

`src/octahedron.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

/// Octahedron with vertices on the coordinate axes.
pub fn octahedron() -> PolygonMesh {

    //PLACE STEP 1-4 HERE

}
```

#### Step 1: Define vertex positions
```rust
    let positions = vec![
        Point3::new(-1.0, 0.0, 0.0), // (-X) [0]
        Point3::new(1.0, 0.0, 0.0),  // (+X) [1]
        Point3::new(0.0, -1.0, 0.0), // (-Y) [2]
        Point3::new(0.0, 1.0, 0.0),  // (+Y) [3]
        Point3::new(0.0, 0.0, -1.0), // (-Z) [4]
        Point3::new(0.0, 0.0, 1.0),  // (+Z) [5]
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
        [0, 4, 2],
        [2, 4, 1],
        [1, 4, 3],
        [3, 4, 0],
        [0, 2, 5],
        [2, 1, 5],
        [1, 3, 5],
        [3, 0, 5],
    ]);
```

#### Step 4: Construct the mesh

```rust
    PolygonMesh::new(attrs, faces)
```

## Export the octahedron

Add `examples/octahedron.rs`:

```rust
fn main() {
    let mesh = truck_meshes::octahedron();
    truck_meshes::write_polygon_mesh(&mesh, "output/octahedron.obj");
}
```

Run it:

```bash
cargo run --example octahedron
```

## View it

Open `output/octahedron.obj` in your preferred OBJ file viewer. You should see a single octahedron.

*gif below from Bambu Studio.*

![Octahedron](images/octahedron.gif)


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
│  └─ octahedron.rs
├─ examples/
│  ├─ triangle.rs
│  ├─ square.rs
│  ├─ tetrahedron.rs
│  ├─ hexahedron.rs
│  └─ octahedron.rs
└─ output/          # exported OBJ files (e.g., output/octahedron.obj)
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

pub mod tetrahedron;
pub use tetrahedron::tetrahedron;

pub mod hexahedron;
pub use hexahedron::hexahedron;

pub mod octahedron;
pub use octahedron::octahedron;
```

`src/octahedron.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

pub fn octahedron() -> PolygonMesh {
    let positions = vec![
        Point3::new(-1.0, 0.0, 0.0),
        Point3::new(1.0, 0.0, 0.0),
        Point3::new(0.0, -1.0, 0.0),
        Point3::new(0.0, 1.0, 0.0),
        Point3::new(0.0, 0.0, -1.0),
        Point3::new(0.0, 0.0, 1.0),
    ];

    let attrs = StandardAttributes {
        positions,
        ..Default::default()
    };

    let faces = Faces::from_iter([
        [0, 4, 2],
        [2, 4, 1],
        [1, 4, 3],
        [3, 4, 0],
        [0, 2, 5],
        [2, 1, 5],
        [1, 3, 5],
        [3, 0, 5],
    ]);

    PolygonMesh::new(attrs, faces)
}
```

`examples/octahedron.rs`:

```rust
fn main() {
    let mesh = truck_meshes::octahedron();
    truck_meshes::write_polygon_mesh(&mesh, "output/octahedron.obj");
}
```

</details>
