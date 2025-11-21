# Normals

Add [normals](https://en.wikipedia.org/wiki/Normal_(geometry)) to meshes in Truck so lighting and shading look correct.

## Why normals matter

- Control light response and perceived smoothness
- Prevent inside-out lighting or flat-looking surfaces
- Necessary for consistent rendering in WebGPU/Vulkan/OpenGL

## Face vs. vertex normals

- **Face normals**: one per polygon; faceted look; good for mechanical shapes.
- **Vertex normals**: one per vertex (averaged from adjacent faces); smooth look; good for curved surfaces.

Truck supports both.

## APIs

Normals live in `StandardAttributes`. Common helpers from `truck_meshalgo::filters`:

- `complete_face_normals(&mesh)`
- `complete_vertex_normals(&mesh)`

They return new meshes with missing normals filled in.

## Example: cube with both normal types

```rust
use truck_meshalgo::prelude::*;
use truck_meshalgo::filters::*;

fn main() {
    let cube = hexahedron();

    let cube_face = complete_face_normals(&cube);
    let cube_vertex = complete_vertex_normals(&cube);

    write_obj(&cube_face, "cube-face-normals.obj");
    write_obj(&cube_vertex, "cube-vertex-normals.obj");
}

fn write_obj(mesh: &PolygonMesh, path: &str) {
    let mut file = std::fs::File::create(path).unwrap();
    obj::write(mesh, &mut file).unwrap();
}
```

Visual difference:

- Face normals → sharp, faceted cube
- Vertex normals → smooth shading (geometry unchanged)

## How Truck computes them

- Face normals: cross product of two edges, normalized, attached per face.
- Vertex normals: average adjacent face normals (angle/area weighting), stored per vertex.

## When to recalc normals

- After changing vertex positions or faces
- After merging/simplifying meshes
- After converting from B-rep/CSG to mesh
- When input meshes have missing/incorrect normals
