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

## How we’ll modularize (step by step)

### 1) **Create new directories**

From the root directory of the project (`truck_user_book/`)

- Run:

```sh 
mkdir -p src/shapes src/utils
```

### 2) **Move shape modules**

- Run:

```sh 
mv src/triangle.rs src/square.rs src/tetrahedron.rs src/hexahedron.rs src/octahedron.rs src/dodecahedron.rs src/icosahedron.rs src/shapes/
```

### 3) **Add `src/shapes/mod.rs`**

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

### 4) **Add the files for the normals helper module and functions**

*(we will insert the code for these on the next page)*

- Add `src/utils/mod.rs`:

```sh
touch src/utils/mod.rs
```

- Add `src/utils/normal_helpers.rs`.

```sh 
touch src/utils/normal_helpers.rs
```

### 5) **Slim `src/lib.rs`**

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
```

#### On the next page, we’ll cover the normals helper functions, and how to wire them into our program.

<br>

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

</details>
