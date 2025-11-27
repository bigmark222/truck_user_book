# Initial Project Setup

Use one library crate (`truck_meshes`) where each shape lives in its own file and is re-exported from `src/lib.rs`.

## 1) Create the workspace

#### Open your terminal:
(Windows users should use **PowerShell**)

```bash
cargo new --lib truck_meshes
cd truck_meshes
mkdir -p examples
mkdir -p output
```

Once you’ve `cd`’d into your new Rust project, launch your editor from that terminal so you’re editing the files you just created (e.g., run `code .` for [VS Code](https://code.visualstudio.com)).

## 2) Add dependencies

Inside the `Cargo.toml` file:

```toml
[dependencies]
truck-meshalgo = "0.4.0"
```

## 3) Seed `src/lib.rs`

Replace the default `src/lib.rs` contents from `cargo new` with this helper (`write_polygon_mesh`), then keep your module re-exports right under it as you add shapes throughout the chapter.

```rust
use truck_meshalgo::prelude::*;

/// Write any mesh to an OBJ file.
pub fn write_polygon_mesh(mesh: &PolygonMesh, path: &str) {
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(mesh, &mut obj).unwrap();
}

// Add shapes here as modules (one file per shape)
// pub mod triangle;
// pub use triangle::triangle;
```

<details>
<summary>File tree:</summary>

```
truck_meshes/
├─ Cargo.toml
├─ src/
│  └─ lib.rs
├─ examples/
│  └─ 
└─ output/   # exported OBJ files live here (create once)
```

</details>

<details>
<summary>Full code (`src/lib.rs`):</summary>

```rust
use truck_meshalgo::prelude::*;

/// Write any mesh to an OBJ file.
pub fn write_polygon_mesh(mesh: &PolygonMesh, path: &str) {
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(mesh, &mut obj).unwrap();
}

// Add shapes here as modules (one file per shape)
// pub mod triangle;
// pub use triangle::triangle;
```

</details>

Keep `write_polygon_mesh` generic—pass the path you want (e.g., `output/triangle.obj`) from each example rather than hardcoding the output folder inside the helper.
