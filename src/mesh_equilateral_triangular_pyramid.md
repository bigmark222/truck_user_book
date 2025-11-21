# Equilateral Triangular Pyramid

Let's build an equilateral triangular pyramid (a [tetrahedron](https://en.wikipedia.org/wiki/Tetrahedron)) using vertex lists and face indices.

Reference: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter2/src/section2_2.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter2/src/section2_2.rs)

## Pyramid mesh

```rust
/// Create an equilateral triangular pyramid
fn trigonal_pyramid() -> PolygonMesh { 

    // STEPS 1-4 GO IN HERE
    
}
```

### Step 1 — vertex positions (base then apex)
```rust

    let positions = vec![
        Point3::new(0.0, 0.0, 0.0),
        Point3::new(1.0, 0.0, 0.0),
        Point3::new(0.5, f64::sqrt(3.0) / 2.0, 0.0),
        Point3::new(0.5, f64::sqrt(3.0) / 6.0, f64::sqrt(6.0) / 3.0),
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
        [2, 1, 0],   // base
        [0, 1, 3],   // side 1
        [1, 2, 3],   // side 2
        [2, 0, 3],   // side 3
    ]);
```
<details>
<summary><strong>Face orientation (important)</strong></summary>

#### Face orientation = the order you list a triangle’s vertices.

- CCW order (counter-clockwise) → face points toward you

- CW order (clockwise) → face points away from you

Rendering engines often hide back faces, so a reversed order can make a face look inside-out or invisible.

#### Rule:

##### When looking at the outside of your shape, list triangle vertices counter-clockwise.

That’s all you need.

![Triangle face winding order in Unity](images/winding-order=triangle-unity.png)

</details>

### Step 4 — build the mesh
```rust
    PolygonMesh::new(attrs, faces)
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
    write_polygon(&trigonal_pyramid(), "trigonal-pyramid.obj");
}
```

#### Run:

```bash
cargo run
```

