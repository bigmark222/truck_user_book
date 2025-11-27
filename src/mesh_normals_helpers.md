# Normal Helper Module

Truck does not provide built-in utilities for computing mesh normals, so here are the helpers we add to keep normals correct across shapes and filters.

## Implement Functions in `src/utils/normal_helpers.rs`

### Import dependencies

```rust
use truck_meshalgo::filters::NormalFilters;
use truck_meshalgo::prelude::*;
```

### `compute_face_normals`

```rust
/// Returns one normalized face normal per polygon.
pub fn compute_face_normals(mesh: &PolygonMesh) -> Vec<Vector3> {
    let positions = mesh.positions();
    mesh.face_iter()
        .map(|face| {
            if face.len() < 3 {
                return Vector3::zero();
            }
            let p0 = positions[face[0].pos];
            let p1 = positions[face[1].pos];
            let p2 = positions[face[2].pos];
            let n = (p1 - p0).cross(p2 - p0);
            if n.magnitude2() < 1e-12 {
                return Vector3::zero();
            }
            n.normalize()
        })
        .collect()
}
```

<details>
<summary>What it does + example</summary>

Computes one unit normal per face using the first three vertices. Degenerate faces return the zero vector so downstream code can decide how to handle them.

```rust
let mesh = build_mesh();
let face_normals = compute_face_normals(&mesh);
mesh.attributes_mut().add_face_normals(face_normals);
```
</details>

### `compute_vertex_normals`

```rust
/// Returns one normalized vertex normal per vertex (averaged from faces).
pub fn compute_vertex_normals(mesh: &PolygonMesh) -> Vec<Vector3> {
    let positions = mesh.positions();
    let face_normals = compute_face_normals(mesh);

    let mut vertex_normals = vec![Vector3::zero(); positions.len()];
    for (face_idx, face) in mesh.face_iter().enumerate() {
        let n = face_normals[face_idx];
        for v in face {
            vertex_normals[v.pos] += n;
        }
    }
    for n in vertex_normals.iter_mut() {
        *n = n.normalize();
    }
    vertex_normals
}
```

<details>
<summary>What it does + example</summary>

Accumulates each face normal into its vertices, then normalizes per-vertex for smooth shading.

```rust
let mesh = load_mesh();
let vertex_normals = compute_vertex_normals(&mesh);
let mut mesh = mesh;
mesh.attributes_mut().add_vertex_normals(vertex_normals);
```
</details>

### `add_face_normals`

```rust
/// Computes and attaches face normals to the mesh.
pub fn add_face_normals(mesh: &mut PolygonMesh) {
    mesh.add_naive_normals(true);
}
```

<details>
<summary>What it does + example</summary>

Convenience wrapper that computes and writes per-face normals in-place.

```rust
let mut mesh = build_mesh();
add_face_normals(&mut mesh);
write_polygon_mesh(&mesh, "output/with-face-normals.obj");
```
</details>

### `add_vertex_normals`

```rust
/// Computes and attaches vertex normals to the mesh.
pub fn add_vertex_normals(mesh: &mut PolygonMesh) {
    mesh.add_smooth_normals(0.8, true);
}
```

<details>
<summary>What it does + example</summary>

Computes smooth per-vertex normals and stores them on the mesh for shading/export.

```rust
let mut mesh = generate_mesh();
add_vertex_normals(&mut mesh);
normalize_vertex_normals(&mut mesh); // optional cleanup
```
</details>

### `normalize_vertex_normals`

```rust
/// Normalizes any existing vertex normals in-place.
pub fn normalize_vertex_normals(mesh: &mut PolygonMesh) {
    mesh.normalize_normals();
}

```

<details>
<summary>What it does + example</summary>

Renormalizes existing vertex normals without changing direction; no-op if none exist.

```rust
let mut mesh = mesh_with_normals();
// ... edit or blend normals ...
normalize_vertex_normals(&mut mesh);
```
</details>

### Add the following re-exports to `src/utils/mod.rs`:

```rust
pub mod normal_helpers;
pub use normal_helpers::{
    compute_face_normals,
    compute_vertex_normals,
    add_face_normals,
    add_vertex_normals,
    normalize_vertex_normals,
};
```
### Add ____ to src/lib.rs

```rust
pub mod utils;
pub use utils::normal_helpers::{
    compute_face_normals,
    compute_vertex_normals,
    add_face_normals,
    add_vertex_normals,
    normalize_vertex_normals,
};
```


## Verify everything works now with `cargo check`

### Run 

```sh
cargo check
```

#### Next page, we will apply these functions to an Icosahedron.


<details>
<summary>Project layout (modular)</summary>

```
truck_meshes/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs
│  ├─ shapes/
│  │  ├─ mod.rs
│  │  ├─ triangle.rs
│  │  ├─ square.rs
│  │  ├─ tetrahedron.rs
│  │  ├─ hexahedron.rs
│  │  ├─ octahedron.rs
│  │  ├─ dodecahedron.rs
│  │  └─ icosahedron.rs
│  └─ utils/
│     ├─ mod.rs
│     └─ normal_helpers.rs
└─ examples/
   ├─ shapes/
   │  ├─ triangle.rs
   │  ├─ square.rs
   │  ├─ tetrahedron.rs
   │  ├─ hexahedron.rs
   │  ├─ octahedron.rs
   │  ├─ dodecahedron.rs
   │  └─ icosahedron.rs
   └─ normals/
      └─ icosahedron.rs
```
</details>

<details>
<summary>Full `src/utils/normal_helpers.rs`</summary>

```rust
use truck_polymesh::prelude::*;

/// Returns one normalized face normal per polygon.
pub fn compute_face_normals(mesh: &PolygonMesh) -> Vec<Vector3> {
    let positions = mesh.positions();
    mesh.faces()
        .iter()
        .map(|face| {
            if face.len() < 3 {
                return Vector3::zero();
            }
            let p0 = positions[face[0]];
            let p1 = positions[face[1]];
            let p2 = positions[face[2]];
            let n = (p1 - p0).cross(p2 - p0);
            if n.magnitude2() < 1e-12 {
                return Vector3::zero();
            }
            n.normalize()
        })
        .collect()
}

/// Returns one normalized vertex normal per vertex (averaged from faces).
pub fn compute_vertex_normals(mesh: &PolygonMesh) -> Vec<Vector3> {
    let positions = mesh.positions();
    let faces = mesh.faces();
    let face_normals = compute_face_normals(mesh);

    let mut vertex_normals = vec![Vector3::zero(); positions.len()];
    for (face_idx, face) in faces.iter().enumerate() {
        let n = face_normals[face_idx];
        for &v in face {
            vertex_normals[v] += n;
        }
    }
    for n in vertex_normals.iter_mut() {
        *n = n.normalize();
    }
    vertex_normals
}

/// Computes and attaches face normals to the mesh.
pub fn add_face_normals(mesh: &mut PolygonMesh) {
    let normals = compute_face_normals(mesh);
    mesh.attributes_mut().add_face_normals(normals);
}

/// Computes and attaches vertex normals to the mesh.
pub fn add_vertex_normals(mesh: &mut PolygonMesh) {
    let normals = compute_vertex_normals(mesh);
    mesh.attributes_mut().add_vertex_normals(normals);
}

/// Normalizes any existing vertex normals in-place.
pub fn normalize_vertex_normals(mesh: &mut PolygonMesh) {
    if let Some(normals) = mesh.attributes_mut().normal_mut() {
        for n in normals.iter_mut() {
            *n = n.normalize();
        }
    }
}
```
</details>
