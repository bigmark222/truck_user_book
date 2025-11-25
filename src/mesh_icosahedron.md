# Icosahedron

An icosahedron is the [dual](https://en.wikipedia.org/wiki/Dual_polyhedron) of a dodecahedron: each dodecahedron face center becomes a vertex; the three centers around each original dodecahedron vertex become one triangle.

Steps:
- Find every dodecahedron face center.
- [Normalize](https://en.wikipedia.org/wiki/Normal_(geometry)) those centers to the [unit sphere](https://en.wikipedia.org/wiki/Unit_sphere) → 20 vertices.
- Group the three centers around each original dodecahedron vertex (corner) → 20 faces.

![Icosahedron from dodecahedron illustration](images/icosa_from_dodeca.png)
##### The yellow icosahedron appears if you remove the orange reference dodecahedron.

## Add the icosahedron module

`src/lib.rs` additions:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

// ...keep earlier functions through dodecahedron...
pub mod icosahedron;
pub use icosahedron::icosahedron;
```
## Construct Main Function

`src/icosahedron.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

// Uses dodecahedron() from the same crate

/// Icosahedron via dual of a dodecahedron.
pub fn icosahedron() -> PolygonMesh {

    //PLACE STEP 1-5 HERE

}
```

#### Step 1: Start from a dodecahedron
```rust
    let dodeca: PolygonMesh = dodecahedron();
    let d_positions = dodeca.positions();
```
#### Step 2: Build vertex positions from face centroids
```rust
    let positions: Vec<Point3> = dodeca
        .face_iter()
        .map(|face| {
            let centroid = face
                .iter()
                .map(|v| d_positions[v.pos].to_vec())
                .sum::<Vector3>();
            Point3::from_vec(centroid.normalize())
        })
        .collect();
```
#### Step 3: Build faces by collecting touching centroids
```rust
    let mut faces: Faces = (0..20)
        .map(|i| {
            dodeca
                .face_iter()
                .enumerate()
                .filter(|(_, f)| f.contains(&i.into()))
                .map(|(idx, _)| idx)
                .collect::<Vec<usize>>()
        })
        .collect();
```
#### Step 4: Fix winding so normals point outward
```rust
    faces.face_iter_mut().for_each(|face| {
        let p: Vec<Point3> = face.iter().map(|v| positions[v.pos]).collect();
        let center = p[0].to_vec() + p[1].to_vec() + p[2].to_vec();
        let normal = (p[1] - p[0]).cross(p[2] - p[0]).normalize();
        if center.dot(normal) < 0.0 {
            face.swap(0, 1);
        }
    });
```
#### Step 5: Construct the mesh
```rust
    PolygonMesh::new(
        StandardAttributes {
            positions,
            ..Default::default()
        },
        faces,
    )
```

## Export the icosahedron

Add `examples/icosahedron.rs`:

```rust
fn main() {
    let mut mesh = truck_meshes::icosahedron();
    mesh.add_naive_normals(true); // optional, for shading
    truck_meshes::write_polygon_mesh(&mesh, "output/icosahedron.obj");
}
```

Run it:

```bash
cargo run --example icosahedron
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
│  ├─ dodecahedron.rs
│  └─ icosahedron.rs
├─ examples/
│  ├─ triangle.rs
│  ├─ square.rs
│  ├─ tetrahedron.rs
│  ├─ hexahedron.rs
│  ├─ octahedron.rs
│  ├─ dodecahedron.rs
│  └─ icosahedron.rs
└─ output/          # exported OBJ files (e.g., output/icosahedron.obj)
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

pub mod icosahedron;
pub use icosahedron::icosahedron;
```

`src/icosahedron.rs`:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

pub fn icosahedron() -> PolygonMesh {
    let dodeca: PolygonMesh = dodecahedron();
    let d_positions = dodeca.positions();

    let positions: Vec<Point3> = dodeca
        .face_iter()
        .map(|face| {
            let centroid = face
                .iter()
                .map(|v| d_positions[v.pos].to_vec())
                .sum::<Vector3>();
            Point3::from_vec(centroid.normalize())
        })
        .collect();

    let mut faces: Faces = (0..20)
        .map(|i| {
            dodeca
                .face_iter()
                .enumerate()
                .filter(|(_, f)| f.contains(&i.into()))
                .map(|(idx, _)| idx)
                .collect::<Vec<usize>>()
        })
        .collect();

    faces.face_iter_mut().for_each(|face| {
        let p: Vec<Point3> = face.iter().map(|v| positions[v.pos]).collect();
        let center = p[0].to_vec() + p[1].to_vec() + p[2].to_vec();
        let normal = (p[1] - p[0]).cross(p[2] - p[0]).normalize();
        if center.dot(normal) < 0.0 {
            face.swap(0, 1);
        }
    });

    PolygonMesh::new(
        StandardAttributes {
            positions,
            ..Default::default()
        },
        faces,
    )
}
```

`examples/icosahedron.rs`:

```rust
fn main() {
    let mut mesh = truck_meshes::icosahedron();
    mesh.add_naive_normals(true);
    truck_meshes::write_polygon_mesh(&mesh, "output/icosahedron.obj");
}
```

</details>
