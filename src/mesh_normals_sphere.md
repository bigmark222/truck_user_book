# Normals - Sphere

Vertex normals are what make a low-poly sphere look smooth instead of faceted. Here we’ll inflate a cube into a sphere, assign per-vertex normals, and export an OBJ.

## Why vertex normals change the look

- **Face normals only:** every quad or triangle shades independently; spheres look like mirror balls.
- **Vertex normals:** store one normal per vertex, let the viewer interpolate across the surface; shading appears round.

If a mesh ships without normals, viewers guess differently (Windows/ParaView use face normals; macOS averages adjacent faces). Providing normals removes that guesswork.

## Full example: cube → sphere with normals

Reuse the shared `write_polygon_mesh` helper and `hexahedron()` shape from `lib.rs` to keep the sample lean:

```rust
use std::iter::FromIterator;
use truck_meshalgo::prelude::*;
use truck_meshes::{hexahedron, write_polygon_mesh};

const DIVISION: usize = 8; // subdivide each cube edge

/// Subdivide the cube, project to a unit sphere, and attach vertex normals.
fn sphere_with_normals() -> PolygonMesh {
    let hexa = hexahedron();

    // Subdivide each cube face into a grid and project points to the unit sphere.
    let positions: Vec<Point3> = hexa
        .face_iter()
        .flat_map(|face| {
            let v: Vec<Vector3> = face
                .iter()
                .map(|vertex| hexa.positions()[vertex.pos].to_vec())
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

    // For a unit sphere, the position direction is the normal.
    let normals: Vec<Vector3> = positions.iter().copied().map(Point3::to_vec).collect();

    // Pair positions with normals.
    let attrs = StandardAttributes { positions, normals, ..Default::default() };

    // Faces reference both position and normal indices (they match here).
    let faces: Faces = (0..6)
        .flat_map(|face_idx| {
            let base = face_idx * (DIVISION + 1) * (DIVISION + 1);

            let to_index = move |i: usize, j: usize| {
                let idx = base + (DIVISION + 1) * i + j;
                (idx, None, Some(idx)) // (position, texture, normal)
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

    PolygonMesh::new(attrs, faces)
}

fn main() {
    let sphere = sphere_with_normals();          // build subdivided, normalized sphere mesh
    write_polygon_mesh(&sphere, "sphere.obj");   // export as OBJ for inspection
}
```

## What to look for

- The OBJ contains per-vertex normals, so most viewers will render it smoothly.
- Because positions and normals share indices, the mesh stays compact; if you ever vary normals independently, keep the separate indices pattern shown above.
- Increase `DIVISION` for a denser sphere; shading stays smooth because normals are normalized unit vectors. 

<details>
<summary>Updated directory layout</summary>

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
      ├─ icosahedron.rs
      └─ sphere.rs
```
</details>
