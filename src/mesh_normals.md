# Normals

Normals tell the renderer which way a surface is facing so lighting and shading look correct. They can be stored per face (flat shading) or per vertex (smooth shading).

## Why normals matter

- Control light response and perceived smoothness
- Prevent inside-out lighting or flat-looking surfaces
- Keep appearance consistent across viewers and exporters

## Face vs. vertex normals

- **Face normals:** one per polygon; computed from the cross product of two edges; yields a sharp, faceted look.
- **Vertex normals:** one per vertex; averaged from adjacent faces; yields smooth shading even on low-poly geometry.

## If normals are missing

Different viewers guess differently:

- Windows and ParaView: use face normals (flat)
- macOS viewer: averages face normals (smooth)

Supply normals yourself to avoid inconsistent results.

## Helper APIs

Normals live inside a mesh's [`StandardAttributes`](https://docs.rs/truck-polymesh/latest/truck_polymesh/struct.StandardAttributes.html#impl-StandardAttributes). Add the local helpers from `utils/normal_helpers.rs` to get raw vectors:

- `compute_face_normals(&mesh)` → one per polygon
- `compute_vertex_normals(&mesh)` → one per vertex
- `add_face_normals(&mut mesh)` → compute + attach per-face normals
- `add_vertex_normals(&mut mesh)` → compute + attach per-vertex normals
- `normalize_vertex_normals(&mut mesh)` → normalize any existing vertex normals

Use these helpers to populate or fix normals before export/rendering.

## Example: cube normals

```rust
use truck_meshalgo::prelude::*;
use truck_meshes::{compute_face_normals, compute_vertex_normals, hexahedron};

fn main() {
    let cube = hexahedron();

    let face_normals = compute_face_normals(&cube);
    let vertex_normals = compute_vertex_normals(&cube);

    // Insert these vectors into the cube's StandardAttributes
    // (face normals per polygon, vertex normals per vertex)
    // before exporting or rendering.
    println!("faces: {} normals", face_normals.len());
    println!("verts: {} normals", vertex_normals.len());
}
```

To mutate in-place instead, call `add_face_normals(&mut mesh)` or `add_vertex_normals(&mut mesh)`, then `normalize_vertex_normals(&mut mesh)` if needed.

## Example: sphere with vertex normals

1) Build a cube:

```rust
fn hexahedron() -> PolygonMesh {
    use truck_meshalgo::prelude::*;
    use std::iter::FromIterator;

    let a = f64::sqrt(3.0) / 3.0;
    let positions = vec![
        Point3::new(-a, -a, -a),
        Point3::new( a, -a, -a),
        Point3::new( a,  a, -a),
        Point3::new(-a,  a, -a),
        Point3::new(-a, -a,  a),
        Point3::new( a, -a,  a),
        Point3::new( a,  a,  a),
        Point3::new(-a,  a,  a),
    ];

    let attrs = StandardAttributes { positions, ..Default::default() };
    let faces = Faces::from_iter([
        [3, 2, 1, 0],
        [0, 1, 5, 4],
        [1, 2, 6, 5],
        [2, 3, 7, 6],
        [3, 0, 4, 7],
        [4, 5, 6, 7],
    ]);

    PolygonMesh::new(attrs, faces)
}
```

2) Subdivide each face, project to a sphere, and gather positions:

```rust
const DIVISION: usize = 8;
let cube = hexahedron();

let positions: Vec<Point3> = cube
    .face_iter()
    .flat_map(|face| {
        let v: Vec<Vector3> = face
            .iter()
            .map(|vertex| cube.positions()[vertex.pos].to_vec())
            .collect();

        (0..=DIVISION)
            .flat_map(move |i| (0..=DIVISION).map(move |j| (i, j)))
            .map(move |(i, j)| {
                let s = i as f64 / DIVISION as f64;
                let t = j as f64 / DIVISION as f64;

                let p =
                    v[0] * (1.0 - s) * (1.0 - t) +
                    v[1] * s         * (1.0 - t) +
                    v[3] * (1.0 - s) * t +
                    v[2] * s         * t;

                Point3::from_vec(p.normalize())
            })
    })
    .collect();
```

3) Assign vertex normals (for a unit sphere, the position direction is the normal):

```rust
let normals: Vec<Vector3> = positions.iter().copied().map(Point3::to_vec).collect();
```

4) Build attributes and faces:

```rust
let attrs = StandardAttributes { positions, normals, ..Default::default() };

let faces: Faces = (0..6)
    .flat_map(|face_idx| {
        let base = face_idx * (DIVISION + 1) * (DIVISION + 1);

        let to_index = move |i: usize, j: usize| {
            let idx = base + (DIVISION + 1) * i + j;
            (idx, None, Some(idx)) // (position index, texture index, normal index)
        };

        (0..DIVISION)
            .flat_map(move |i| (0..DIVISION).map(move |j| (i, j)))
            .map(move |(i, j)| {
                [
                    to_index(i, j),
                    to_index(i + 1, j),
                    to_index(i + 1, j + 1),
                    to_index(i, j + 1),
                ]
            })
    })
    .collect();
```

5) Export:

```rust
fn write_polygon_mesh(mesh: &PolygonMesh, path: &str) {
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(mesh, &mut obj).unwrap();
}

let sphere = PolygonMesh::new(attrs, faces);
write_polygon_mesh(&sphere, "sphere.obj");
```

## When to recompute normals

- After changing vertex positions or faces
- After merging/simplifying meshes
- After converting from B-rep/CSG to mesh
- When input meshes have missing or incorrect normals
