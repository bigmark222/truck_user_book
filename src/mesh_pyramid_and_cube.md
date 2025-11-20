# Triangular Pyramid and Cube

Build two slightly more complex meshes: an equilateral triangular pyramid (tetrahedron) and a cube. These examples show how to construct multi-face objects using vertex lists and face index lists.

Reference code: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter2/src/section2_2.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter2/src/section2_2.rs)

## Equilateral triangular pyramid

```rust
/// Create an equilateral triangular pyramid
fn trigonal_pyramid() -> PolygonMesh {
    // Step 1 — vertex positions (base then apex)
    let positions = vec![
        Point3::new(0.0, 0.0, 0.0),
        Point3::new(1.0, 0.0, 0.0),
        Point3::new(0.5, f64::sqrt(3.0) / 2.0, 0.0),
        Point3::new(0.5, f64::sqrt(3.0) / 6.0, f64::sqrt(6.0) / 3.0),
    ];

    // Step 2 — attributes (positions only)
    let attrs = StandardAttributes {
        positions,
        ..Default::default()
    };

    // Step 3 — faces (0-based indices)
    let faces = Faces::from_iter([
        [2, 1, 0],   // base
        [0, 1, 3],   // side 1
        [1, 2, 3],   // side 2
        [2, 0, 3],   // side 3
    ]);

    // Step 4 — build the mesh
    PolygonMesh::new(attrs, faces)
}
```

Index mapping:

```
      3
     /|\
    / | \
   2--+--1
    \ | /
     \|/
      0
```

## Cube

A cube has 8 vertices and 6 quad faces.

```rust
/// Create a cube
fn cube() -> PolygonMesh {
    let positions = vec![
        Point3::new(0.0, 0.0, 0.0),
        Point3::new(1.0, 0.0, 0.0),
        Point3::new(1.0, 1.0, 0.0),
        Point3::new(0.0, 1.0, 0.0),
        Point3::new(0.0, 0.0, 1.0),
        Point3::new(1.0, 0.0, 1.0),
        Point3::new(1.0, 1.0, 1.0),
        Point3::new(0.0, 1.0, 1.0),
    ];

    let attrs = StandardAttributes {
        positions,
        ..Default::default()
    };

    let faces = Faces::from_iter([
        [3, 2, 1, 0], // bottom
        [0, 1, 5, 4], // front
        [1, 2, 6, 5], // right
        [2, 3, 7, 6], // back
        [3, 0, 4, 7], // left
        [4, 5, 6, 7], // top
    ]);

    PolygonMesh::new(attrs, faces)
}
```

## Outputting the meshes

```rust
/// Output the contents of `polygon` to the file specified by `path`.
fn write_polygon(polygon: &PolygonMesh, path: &str) {
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(polygon, &mut obj).unwrap();
}

fn main() {
    write_polygon(&trigonal_pyramid(), "trigonal-pyramid.obj");
    write_polygon(&cube(), "cube.obj");
}
```

Run:

```bash
cargo run
```

This generates both OBJ files.

## Face orientation (important)

Face orientation is determined by vertex order. For closed shapes, a right-handed screw motion along the vertex order should point outward. Reversing a face (e.g., `[0, 1, 2]` -> `[2, 1, 0]`) can make it render inside-out or invisible in some viewers.
