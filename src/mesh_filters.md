# Mesh Filters

Use Truck’s mesh filters to clean, modify, and analyze meshes before rendering or export.

Reference: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter2/src/section2_5.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter2/src/section2_5.rs)

## Topology conditions

Classify meshes (regular, oriented, closed, etc.):

```rust
let condition = mesh.shell_condition();
println!("{:?}", condition);
```

- **Irregular**: an edge has 3+ faces.
- **Regular**: each edge has at most two faces.
- **Oriented**: no edge appears twice with the same direction.
- **Closed**: every edge appears exactly twice (watertight manifold).

Example (sphere OBJ):

```rust
use truck_meshalgo::prelude::*;

let mut mirror_ball = obj::read(include_bytes!("sphere.obj").as_slice()).unwrap();
println!("default shell condition: {:?}", mirror_ball.shell_condition());
```

## Merge duplicate vertices

Remove seams created by duplicated coordinates:

```rust
// Vertices within 1e-3 are merged
mesh.put_together_same_attrs(1.0e-3);
println!("after merge: {:?}", mesh.shell_condition());
```

## Add normals

Faceted (per-face) normals:

```rust
mesh.add_naive_normals(true); // overwrite existing normals
write_polygon(&mesh, "mirror-ball.obj");
```

Smooth normals (blend across angles):

```rust
// 1.0 rad (~57°) smooths most edges; lower keeps creases
mesh.add_smooth_normals(1.0, true);
write_polygon(&mesh, "mirror-ball-with-smooth-normal.obj");
```

## When to use which

- `put_together_same_attrs`: importing OBJ/patch models, fixing seams, removing duplicate geometry.
- `add_naive_normals(true)`: sharp mechanical parts, crisp reflections, debugging face orientation.
- `add_smooth_normals(angle, true)`: spheres or organic shapes needing soft shading.

## Save and run

Use the existing `write_polygon` helper to export OBJ files, then:

```bash
cargo run
```

Inspect the results in an OBJ viewer (Preview, Blender, ParaView) to verify topology and normals.
