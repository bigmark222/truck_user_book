# First Triangle


## Add the triangle module to lib.rs

`src/lib.rs` (root exports + helper):

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

/// Write any mesh to an OBJ file.
pub fn write_polygon_mesh(mesh: &PolygonMesh, path: &str) {
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(mesh, &mut obj).unwrap();
}

pub mod triangle; //add this
pub use triangle::triangle; //add this
```
## Construct Main Function

`src/triangle.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

/// A single equilateral triangle in the XY plane.
pub fn triangle() -> PolygonMesh {

    //PLACE STEP 1-4 HERE

}
```
#### Step 1: Define vertex positions
```rust
    let positions = vec![
        Point3::new(0.0, 0.0, 0.0),
        Point3::new(1.0, 0.0, 0.0),
        Point3::new(0.5, f64::sqrt(3.0) / 2.0, 0.0),
    ];
```
<details>
<summary>Explanation</summary>

Create three `Point3` coordinates that form an equilateral triangle on the XY plane. Two points sit on the X axis at y = 0, and the third is lifted to `sqrt(3)/2` so all sides are length 1. These positions are the raw vertex data the mesh will consume.

</details>

#### Step 2: Build attribute set

```rust
    let attrs = StandardAttributes {
        positions,
        ..Default::default()
    };
```
<details>
<summary>Explanation</summary>

Wrap the vertex positions into the `StandardAttributes` container, which is where meshes expect per-vertex data (positions, normals, UVs, etc.). We only set positions and leave every other attribute at its default.

</details>

#### Step 3: Define mesh faces

```rust
    let faces = Faces::from_iter([[0, 1, 2]]);
```
<details>
<summary>Explanation</summary>

Specify the triangle’s topology by listing vertex indices. The single face references vertices 0, 1, and 2 in counter-clockwise order, which sets the face normal to point along +Z.

</details>

#### Step 4: Construct the mesh

```rust
    PolygonMesh::new(attrs, faces)

```
<details>
<summary>Explanation</summary>

Assemble the mesh by pairing the attribute data with the face list. The returned `PolygonMesh` is ready to render or export (e.g., via `write_polygon_mesh` to an OBJ file).

</details>

## Export the triangle

Add a tiny example at `examples/triangle.rs`:

```rust
fn main() {
    let mesh = truck_meshes::triangle();
    truck_meshes::write_polygon_mesh(&mesh, "output/triangle.obj");
}
```

Run it:

```bash
cargo run --example triangle
```

## View it

Open `output/triangle.obj` in Preview/3D Viewer/ParaView/Blender. You should see a single triangle.

<details>
<summary>File tree after this step</summary>

```
truck_meshes/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs
│  └─ triangle.rs
├─ examples/
│  └─ triangle.rs   (optional helper to export)
└─ output/          # exported OBJ files (e.g., output/triangle.obj)
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
```

`src/triangle.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

pub fn triangle() -> PolygonMesh {
    let positions = vec![
        Point3::new(0.0, 0.0, 0.0),
        Point3::new(1.0, 0.0, 0.0),
        Point3::new(0.5, f64::sqrt(3.0) / 2.0, 0.0),
    ];

    let attrs = StandardAttributes {
        positions,
        ..Default::default()
    };

    let faces = Faces::from_iter([[0, 1, 2]]);

    PolygonMesh::new(attrs, faces)
}
```
`examples/triangle.rs`:

```rust
fn main() {
    let mesh = truck_meshes::triangle();
    truck_meshes::write_polygon_mesh(&mesh, "output/triangle.obj");
}
```

</details>
