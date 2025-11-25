# Regular Polyhedrons

Generate Platonic solids using Truck’s built-in helpers instead of hand-writing vertices and faces.

## Available primitives

In `truck_meshalgo::primitive`:

- Tetrahedron
- Hexahedron (cube)
- Octahedron
- Dodecahedron
- Icosahedron

Each call returns a `PolygonMesh` with vertex positions, oriented faces, and consistent topology.

## Example: generate all five

```rust
use truck_meshalgo::prelude::*;
use truck_meshalgo::primitive::*;

fn main() {
    let tetra = tetrahedron();
    let hexa = hexahedron(); // cube
    let octa = octahedron();
    let dodeca = dodecahedron();
    let icosa = icosahedron();

    // Save to OBJ
    write_obj(&tetra, "output/tetrahedron.obj");
    write_obj(&hexa, "output/hexahedron.obj");
    write_obj(&octa, "output/octahedron.obj");
    write_obj(&dodeca, "output/dodecahedron.obj");
    write_obj(&icosa, "output/icosahedron.obj");
}

/// Helper for writing OBJ files
fn write_obj(mesh: &PolygonMesh, path: &str) {
    let mut file = std::fs::File::create(path).unwrap();
    obj::write(mesh, &mut file).unwrap();
}
```

## Scale and position

By default, solids are centered at the origin with unit edge length. Transform them with filters, e.g. scale:

```rust
use truck_meshalgo::filters::scale;

let bigger = scale(&tetra, 3.0);
```

## Why use primitives?

- Quick tests for rendering pipelines and import/export
- Experiments with normals, filters, and transformations
- Building blocks for more complex scenes
- Easy way to learn Truck’s mesh structure

## Viewing the polyhedra

Use any OBJ viewer: Windows 3D Viewer, macOS Preview, Blender, ParaView. ParaView is great for wireframe and face-normal inspection.
