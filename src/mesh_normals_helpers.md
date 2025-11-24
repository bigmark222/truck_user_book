# Normal Helper Module

Truck does not provide built-in utilities for computing mesh normals, so here are the helpers we add to keep normals correct across shapes and filters.

## Implement Functions in `src/utils/normal_helpers.rs`

### Import dependencies

```rust
use truck_polymesh::prelude::*;
```

### `compute_face_normals`

```rust
/// Returns one normalized face normal per polygon.
pub fn compute_face_normals(mesh: &PolygonMesh) -> Vec<Vector3> {
    let positions = mesh.positions();
    mesh.faces()
        .iter()
        .map(|face| {
            // Need at least 3 vertices to define a plane
            if face.len() < 3 {
                return Vector3::zero();
            }

            // Grab first three points of the face
            let p0 = positions[face[0]];
            let p1 = positions[face[1]];
            let p2 = positions[face[2]];

            // Build two edge vectors
            let e1 = p1 - p0;
            let e2 = p2 - p0;

            // Cross product gives perpendicular direction
            let n = e1.cross(e2);

            // Collinearity / zero-area check
            if n.magnitude2() < 1e-12 {
                return Vector3::zero();
            }

            // Normalize to unit length
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
pub fn compute_vertex_normals(mesh: &PolygonMesh) -> Vec<Vector3> {
    let positions = mesh.positions();              // get all vertex positions
    let faces = mesh.faces();                      // get all faces
    let face_normals = compute_face_normals(mesh); // compute one normal per face

    // one accumulator vector per vertex (start at zero)
    let mut vertex_normals = vec![Vector3::zero(); positions.len()];

    // add each face's normal to all its vertices
    for (face_idx, face) in faces.iter().enumerate() {
        let n = face_normals[face_idx];            // this face's normal
        for &v in face {                           // each vertex in the face
            vertex_normals[v] += n;                // accumulate the normal
        }
    }

    // normalize all accumulated normals
    for n in vertex_normals.iter_mut() {
        *n = n.normalize();                        // convert to unit vectors
    }

    vertex_normals                                 // return one normal per vertex
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
    let normals = compute_face_normals(mesh);
    mesh.attributes_mut().add_face_normals(normals);
}
```

<details>
<summary>What it does + example</summary>

Convenience wrapper that computes and writes per-face normals in-place.

```rust
let mut mesh = build_mesh();
add_face_normals(&mut mesh);
write_polygon_mesh(&mesh, "with-face-normals.obj");
```
</details>

### `add_vertex_normals`

```rust
/// Computes and attaches vertex normals to the mesh.
pub fn add_vertex_normals(mesh: &mut PolygonMesh) {
    let normals = compute_vertex_normals(mesh);
    mesh.attributes_mut().add_vertex_normals(normals);
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
    if let Some(normals) = mesh.attributes_mut().normal_mut() {
        for n in normals.iter_mut() {
            *n = n.normalize();
        }
    }
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
