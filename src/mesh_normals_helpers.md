# Normal Helper Modules

Truck has normal filters, but no helper functions for manually computing face or vertex normals, so we provide our own utilities here.

## Implement Functions in `src/utils/normal_helpers.rs`

### Import dependencies

```rust
use truck_meshalgo::prelude::*;
```

### `compute_face_normals`

```rust
/// Returns one normalized face normal per polygon.
pub fn compute_face_normals(mesh: &PolygonMesh) -> Vec<Vector3> {
    let positions = mesh.positions();
    mesh.face_iter()
        .map(|face| {
            // Need at least three vertices to form a plane
            if face.len() < 3 {
                return Vector3::zero();
            }
            // First three positions define the face plane
            let p0 = positions[face[0].pos];
            let p1 = positions[face[1].pos];
            let p2 = positions[face[2].pos];
            let n = (p1 - p0).cross(p2 - p0);
            // Skip degenerate/collinear faces
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

Derives a normalized face normal from the first three vertices of each polygon. **Degenerate** (cannot form a valid surface) **or collinear** (vertices all lie on the same straight line.) **faces** yield `Vector3::zero()`, enabling later routines to detect and correct them.

<b>Example</b>
```rust
let mut mesh = build_mesh();
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
        // Accumulate adjacent face normals onto each vertex
        for v in face {
            vertex_normals[v.pos] += n;
        }
    }
    // Normalize summed vectors so each vertex normal is unit length
    for n in vertex_normals.iter_mut() {
        *n = n.normalize();
    }
    vertex_normals
}
```

<details>
<summary>What it does + example</summary>

Accumulates the face normals that touch each vertex and normalizes the sum so shared vertices shade smoothly across adjacent polygons.

<b>Example</b>

```rust
let mut mesh = load_mesh();
let vertex_normals = compute_vertex_normals(&mesh);
mesh.attributes_mut().add_vertex_normals(vertex_normals);
```

</details>

### `add_face_normals`

```rust
/// Computes and attaches a normal per face, wiring each vertex in the face to that normal.
pub fn add_face_normals(mesh: &mut PolygonMesh) {
    let normals = compute_face_normals(mesh); // one normal per polygon
    let editor = mesh.editor();
    editor.attributes.normals = normals; // store normals into attributes
    for (idx, face) in editor.faces.face_iter_mut().enumerate() {
        for v in face.iter_mut() {
            v.nor = Some(idx); // point each vertex's normal index to its face normal
        }
    }
}
```

<details>
<summary>What it does + example</summary>

Computes per-face normals with `compute_face_normals` and wires each vertex in a face to that face's normal index.

<b>Example</b>
```rust
let mut mesh = icosahedron();
add_face_normals(&mut mesh);
write_polygon_mesh(&mesh, "output/icosahedron_flat.obj");
```
</details>

### `add_vertex_normals`
```rust 
/// Computes and attaches a normal per vertex, wiring indices to match position indices.
pub fn add_vertex_normals(mesh: &mut PolygonMesh) {
    let normals = compute_vertex_normals(mesh); // smooth normals per vertex
    let editor = mesh.editor();
    editor.attributes.normals = normals; // store normals into attributes
    for face in editor.faces.face_iter_mut() {
        for v in face.iter_mut() {
            v.nor = Some(v.pos); // normal index aligns with vertex position index
        }
    }
}
```

<details>
<summary>What it does + example</summary>

Computes smooth vertex normals using `compute_vertex_normals` and stores them so each vertex reuses its own normal index.

<b>Example</b>
```rust
let mut mesh = icosahedron();
add_vertex_normals(&mut mesh);
write_polygon_mesh(&mesh, "output/icosahedron_smooth.obj");
```
</details>

### `normalize_vertex_normals`

```rust

/// Normalizes any normals currently stored on the mesh in-place.
pub fn normalize_vertex_normals(mesh: &mut PolygonMesh) {
    let editor = mesh.editor();
    for n in editor.attributes.normals.iter_mut() {
        *n = n.normalize(); // keep direction, ensure unit length
    }
}

```

<details>
<summary>What it does + example</summary>

Renormalizes all stored normals in place, leaving their directions unchanged; if no normals exist, nothing happens.

<b>Example</b>
```rust
let mut mesh = mesh_with_normals();
// ... edit or mix normals ...
normalize_vertex_normals(&mut mesh);
```
</details>


### Add the following re-exports to `src/utils/mod.rs`

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

### Add the normal helpers re-exports to `src/lib.rs`

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

#### Next page, we will apply these functions to an Icosahedron


<details>
<summary>Directory layout (modular)</summary>

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
├─ examples/
│  ├─ triangle.rs
│  ├─ square.rs
│  ├─ tetrahedron.rs
│  ├─ hexahedron.rs
│  ├─ octahedron.rs
│  ├─ dodecahedron.rs
│  └─ icosahedron.rs
│  
└─ output/          # exported OBJ files from examples

```
</details>

<details>
<summary>Full code:</summary>

`src/utils/normal_helpers.rs`

```rust
use truck_meshalgo::prelude::*;

/// Returns one normalized face normal per polygon.
pub fn compute_face_normals(mesh: &PolygonMesh) -> Vec<Vector3> {
    let positions = mesh.positions();
    mesh.face_iter()
        .map(|face| {
            // Need at least three vertices to form a plane
            if face.len() < 3 {
                return Vector3::zero();
            }
            // First three positions define the face plane
            let p0 = positions[face[0].pos];
            let p1 = positions[face[1].pos];
            let p2 = positions[face[2].pos];
            let n = (p1 - p0).cross(p2 - p0);
            // Skip degenerate/collinear faces
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
    let face_normals = compute_face_normals(mesh);

    let mut vertex_normals = vec![Vector3::zero(); positions.len()];
    for (face_idx, face) in mesh.face_iter().enumerate() {
        let n = face_normals[face_idx];
        // Accumulate adjacent face normals onto each vertex
        for v in face {
            vertex_normals[v.pos] += n;
        }
    }
    // Normalize summed vectors so each vertex normal is unit length
    for n in vertex_normals.iter_mut() {
        *n = n.normalize();
    }
    vertex_normals
}

/// Computes and attaches a normal per face, wiring each vertex in the face to that normal.
pub fn add_face_normals(mesh: &mut PolygonMesh) {
    let normals = compute_face_normals(mesh);
    let editor = mesh.editor();
    editor.attributes.normals = normals;
    for (idx, face) in editor.faces.face_iter_mut().enumerate() {
        for v in face.iter_mut() {
            v.nor = Some(idx);
        }
    }
}

/// Computes and attaches a normal per vertex, wiring indices to match position indices.
pub fn add_vertex_normals(mesh: &mut PolygonMesh) {
    let normals = compute_vertex_normals(mesh);
    let editor = mesh.editor();
    editor.attributes.normals = normals;
    for face in editor.faces.face_iter_mut() {
        for v in face.iter_mut() {
            v.nor = Some(v.pos);
        }
    }
}

/// Normalizes any normals currently stored on the mesh in-place.
pub fn normalize_vertex_normals(mesh: &mut PolygonMesh) {
    let editor = mesh.editor();
    for n in editor.attributes.normals.iter_mut() {
        *n = n.normalize();
    }
}
```

`src/utils/mod.rs`

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



`src/lib.rs`

```rust
use truck_meshalgo::prelude::*;

pub fn write_polygon_mesh(mesh: &PolygonMesh, path: &str) {
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(mesh, &mut obj).unwrap();
}

pub mod shapes;
pub use shapes::{
    triangle,
    square,
    tetrahedron,
    hexahedron,
    octahedron,
    dodecahedron,
    icosahedron,
};

pub mod utils;
pub use utils::normal_helpers::{
    compute_face_normals,
    compute_vertex_normals,
    add_face_normals,
    add_vertex_normals,
    normalize_vertex_normals,
};
```

</details>
