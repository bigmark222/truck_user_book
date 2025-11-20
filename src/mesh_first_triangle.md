# First Triangle

In this first example, we’ll create the simplest possible mesh: a single triangle. By the end of this section, you will generate an `.obj` file you can view in any 3D viewer.

Reference code: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter2/src/section2_1.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter2/src/section2_1.rs)

## Create a new workspace
This directory will be your working folder for the tutorial. You can name it anything you like.

```bash
cargo new --bin <workspace-name>
cd <workspace-name>
```
<details>
<summary>Example copy/paste cargo project creation</summary>

```bash
cargo new --bin truck_user_tutorial
cd truck_user_tutorial
```

</details>

## Add dependencies

Open your `Cargo.toml` and add the mesh algorithm crate:

```toml
[dependencies]
truck-meshalgo = "0.6.0"
```

This crate contains everything you need to build and export polygon meshes.

## Write the simplest OBJ file

Open `src/main.rs` and replace it with:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

/// Create a mesh containing one equilateral triangle and save it as an OBJ file.
fn main() {
    // Vertex positions
    let positions = vec![
        Point3::new(0.0, 0.0, 0.0),
        Point3::new(1.0, 0.0, 0.0),
        Point3::new(0.5, f64::sqrt(3.0) / 2.0, 0.0),
    ];

    // Register the vertex attributes (positions only in this example)
    let attrs = StandardAttributes {
        positions,
        ..Default::default()
    };

    // Create a single triangular face referencing the 3 vertices above
    let faces = Faces::from_iter([[0, 1, 2]]);

    // Build the mesh
    let polygon = PolygonMesh::new(attrs, faces);

    // Write it to an OBJ file
    let mut obj = std::fs::File::create("triangle.obj").unwrap();
    obj::write(&polygon, &mut obj).unwrap();
}
```

This program constructs:

- 3 vertices
- 1 triangular face
- an OBJ file named `triangle.obj`

## Run the program

From your project directory, run:

```bash
cargo run
```

If everything works, you will see a new file: `triangle.obj`. This uses the Wavefront OBJ format, which is widely supported and easy to view.

## Viewing the triangle

Most operating systems include a built-in OBJ viewer:

- Windows: 3D Viewer
- macOS: Preview

Both are fine for quick checks but limited for edge display and advanced visualization.

### A better viewer (recommended)

[ParaView](https://www.paraview.org/) offers:

- edge display
- surface shading
- advanced rendering
- support for large meshes
- scientific visualization features
