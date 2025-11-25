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
   ├─ shapes/                   # shape-only examples
   │  ├─ triangle.rs
   │  ├─ square.rs
   │  ├─ tetrahedron.rs
   │  ├─ hexahedron.rs
   │  ├─ octahedron.rs
   │  ├─ dodecahedron.rs
   │  └─ icosahedron.rs
   └─ normals/                  # normals + filters walkthroughs
      └─ icosahedron.rs
```

## How we’ll modularize (step by step)

1) **Create folders**
- Run `mkdir -p src/shapes src/utils examples/shapes examples/normals`.

2) **Move shape modules**
- Run `mv src/triangle.rs src/square.rs src/tetrahedron.rs src/hexahedron.rs src/octahedron.rs src/dodecahedron.rs src/icosahedron.rs src/shapes/`.
- Update sibling imports only for `icosahedron`: we re-export shapes from `lib.rs`, so keep `use crate::dodecahedron;` (no other shape modules depend on siblings).

3) **Add `src/shapes/mod.rs`**
- In `src/shapes/mod.rs`, declare and re-export shapes:
```rust
pub mod triangle;
pub use triangle::triangle;
// repeat for square, tetrahedron, hexahedron, octahedron, dodecahedron, icosahedron
```

4) **Move normals helpers**
- Place the helper code in `src/utils/normal_helpers.rs`.
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
- Run `mv examples/triangle.rs examples/square.rs examples/tetrahedron.rs examples/hexahedron.rs examples/octahedron.rs examples/dodecahedron.rs examples/icosahedron.rs examples/shapes/`.
- Put normals/filter walkthroughs under `examples/normals/` (for example, `examples/normals/icosahedron.rs`).
- You don’t need to change the `use truck_meshes::triangle;` style imports in examples because `lib.rs` will keep re-exporting these symbols.

7) **Update your code/docs**
- Adjust code snippets and paths in your project to use `shapes::...` and `utils/normal_helpers.rs`.
- Refresh any file-tree notes or READMEs to match the modular layout.

8) **Verify**
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
│  │  ├─ triangle.rs
│  │  ├─ square.rs
│  │  ├─ tetrahedron.rs
│  │  ├─ hexahedron.rs
│  │  ├─ octahedron.rs
│  │  ├─ dodecahedron.rs
│  │  └─ icosahedron.rs
│  └─ normals/
│     └─ icosahedron.rs
└─ output/          # exported OBJ files from examples
```
</details>

<details>
<summary>Final lib/mod files</summary>

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
</details>
