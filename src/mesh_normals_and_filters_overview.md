# Mesh Normals and Filters

This section explains how to clean up and refine meshes after constructing them from vertices and faces, and how we will reorganize the crate to support normals and filters.

As we scale our project, let's upgrade our directory layout so it can properly handle a little more scale without getting messy.


## Current Directory Layout

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
└─ examples/
   ├─ triangle.rs
   ├─ square.rs
   ├─ tetrahedron.rs
   ├─ hexahedron.rs
   ├─ octahedron.rs
   ├─ dodecahedron.rs
   └─ icosahedron.rs
```

## Target Directory Layout

```
truck_meshes/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs                    # lean: pub mod/pub use only
│  ├─ utils/
│  │  ├─ mod.rs                 # exports helpers
│  │  └─ normal_helpers.rs      # normals helpers live here
│  └─ shapes/                   # each shape in its own module
│     ├─ mod.rs                 # exports shapes
│     ├─ triangle.rs
│     ├─ square.rs
│     ├─ tetrahedron.rs
│     ├─ hexahedron.rs
│     ├─ octahedron.rs
│     ├─ dodecahedron.rs
│     └─ icosahedron.rs
└─ examples/
   ├─ shapes/                   # shape-only examples (per example folder with main.rs)
   │  ├─ triangle/
   │  │  └─ main.rs
   │  ├─ square/
   │  │  └─ main.rs
   │  ├─ tetrahedron/
   │  │  └─ main.rs
   │  ├─ hexahedron/
   │  │  └─ main.rs
   │  ├─ octahedron/
   │  │  └─ main.rs
   │  ├─ dodecahedron/
   │  │  └─ main.rs
   │  └─ icosahedron/
   │     └─ main.rs
   └─ normals/                  # normals + filters walkthroughs
      └─ icosahedron/
         └─ main.rs
```

## How we’ll modularize (step by step)

1) **Create folders**

- From the root directory of the project (`truck_user_book/`)
    - Run:

    ```sh 
    mkdir -p src/shapes src/utils examples/shapes examples/normals
    ```

2) **Move shape modules**

    - Run:

    ```sh 
    mv src/triangle.rs src/square.rs src/tetrahedron.rs src/hexahedron.rs src/octahedron.rs src/dodecahedron.rs src/icosahedron.rs src/shapes/
    ```

3) **Add `src/shapes/mod.rs`**

- In `src/shapes/mod.rs`, declare and re-export shapes:

```rust
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

4) **Move normals helpers**

- Add `src/utils/mod.rs`:

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

- Place the helper code in `src/utils/normal_helpers.rs`.

```rust
use truck_meshalgo::filters::NormalFilters;
use truck_meshalgo::prelude::*;
```

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

```rust
/// Computes and attaches face normals to the mesh.
pub fn add_face_normals(mesh: &mut PolygonMesh) {
    mesh.add_naive_normals(true);
}
```

```rust
/// Computes and attaches vertex normals to the mesh.
pub fn add_vertex_normals(mesh: &mut PolygonMesh) {
    mesh.add_smooth_normals(0.8, true);
}
```

```rust
/// Normalizes any existing vertex normals in-place.
pub fn normalize_vertex_normals(mesh: &mut PolygonMesh) {
    mesh.normalize_normals();
}

```

5) **Slim `src/lib.rs`**

- Keep it lean with modules + re-exports:

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

6) **Move examples**

- Create per-example folders with `main.rs` entrypoints:

```sh
mkdir -p examples/shapes/{triangle,square,tetrahedron,hexahedron,octahedron,dodecahedron,icosahedron}
```

```sh
mkdir -p examples/normals/{icosahedron, sphere}
```

- Move each example into its folder as `main.rs`
- (bash/zsh) version:

```sh
cd examples
while read -r src dst; do
  mkdir -p "$(dirname "$dst")"
  mv "$src" "$dst"
done <<'EOF'
triangle.rs        shapes/triangle/main.rs
square.rs          shapes/square/main.rs
tetrahedron.rs     shapes/tetrahedron/main.rs
hexahedron.rs      shapes/hexahedron/main.rs
octahedron.rs      shapes/octahedron/main.rs
dodecahedron.rs    shapes/dodecahedron/main.rs
icosahedron.rs     shapes/icosahedron/main.rs
EOF
```

- Fish shell version:

```fish
cd examples
for pair in \
  "triangle.rs shapes/triangle/main.rs" \
  "square.rs shapes/square/main.rs" \
  "tetrahedron.rs shapes/tetrahedron/main.rs" \
  "hexahedron.rs shapes/hexahedron/main.rs" \
  "octahedron.rs shapes/octahedron/main.rs" \
  "dodecahedron.rs shapes/dodecahedron/main.rs" \
  "icosahedron.rs shapes/icosahedron/main.rs"
    set src (echo $pair | awk '{print $1}')
    set dst (echo $pair | awk '{print $2}')
    mkdir -p (dirname $dst)
    mv $src $dst
end
```

- You don’t need to change the `use truck_meshes::triangle;` style imports in examples because `lib.rs` will keep re-exporting these symbols.

7) **Verify**

- Run `cargo check` (and `cargo test` if present) to confirm modules and examples still compile.

<details>
<summary>Final file tree (after modularization)</summary>

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
│  ├─ shapes/
│  │  ├─ triangle/
│  │  │  └─ main.rs
│  │  ├─ square/
│  │  │  └─ main.rs
│  │  ├─ tetrahedron/
│  │  │  └─ main.rs
│  │  ├─ hexahedron/
│  │  │  └─ main.rs
│  │  ├─ octahedron/
│  │  │  └─ main.rs
│  │  ├─ dodecahedron/
│  │  │  └─ main.rs
│  │  └─ icosahedron/
│  │     └─ main.rs
│  └─ normals/
│     └─ icosahedron/
│        └─ main.rs
└─ output/          # exported OBJ files from examples
```

</details>

<details>
<summary>Full code:</summary>

`src/lib.rs`

```rust
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

`src/shapes/mod.rs`

```rust
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

`src/utils/normal_helpers.rs`

```rust
use truck_meshalgo::filters::NormalFilters;
use truck_meshalgo::prelude::*;

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

/// Computes and attaches face normals to the mesh.
pub fn add_face_normals(mesh: &mut PolygonMesh) {
    mesh.add_naive_normals(true);
}

/// Computes and attaches vertex normals to the mesh.
pub fn add_vertex_normals(mesh: &mut PolygonMesh) {
    mesh.add_smooth_normals(0.8, true);
}

/// Normalizes any existing vertex normals in-place.
pub fn normalize_vertex_normals(mesh: &mut PolygonMesh) {
    mesh.normalize_normals();
}

```

</details>
