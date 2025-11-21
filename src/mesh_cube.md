# Cube

Now let's build a cube (a [hexahedron](https://en.wikipedia.org/wiki/Hexahedron)) from vertex lists and quad faces.

Reference: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter2/src/section2_2.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter2/src/section2_2.rs)

## Cube mesh

```rust
/// Create a cube
fn cube() -> PolygonMesh {

    //PLACE STEPS 1-4 IN HERE

}
```

### Step 1 — vertex positions

```rust
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
```

### Step 2 — attributes (positions only)

```rust
    let attrs = StandardAttributes {
        positions,
        ..Default::default()
    };
```

### Step 3 — faces (0-based indices)

```rust
    let faces = Faces::from_iter([
        [3, 2, 1, 0], // bottom
        [0, 1, 5, 4], // front
        [1, 2, 6, 5], // right
        [2, 3, 7, 6], // back
        [3, 0, 4, 7], // left
        [4, 5, 6, 7], // top
    ]);
```

### Step 4 — build the mesh

```rust
    PolygonMesh::new(attrs, faces)
```

### Step 5 - Create Write to OBJ Function

```rust
/// Output the contents of `polygon` to the file specified by `path`.
fn write_polygon(polygon: &PolygonMesh, path: &str) {
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(polygon, &mut obj).unwrap();
}
```

### Step 6 - Insert that Function into the Main Function.

```rust
fn main() {
    write_polygon(&cube(), "cube.obj");
}
```

#### Run:

```bash
cargo run
```
