# Mesh Filters

#### Use Truck’s mesh filters to clean, modify, and analyze meshes before rendering or export.

Create `examples/normals_filter_sphere.rs`

```rust
use truck_meshalgo::prelude::*;
use truck_meshes::write_polygon_mesh;

fn main() {

   // PLACE ALL EXAMPLES IN HERE

}
```

## Topology conditions

Classify meshes (regular, oriented, closed, etc.):
<br>
Example (using `sphere.obj` from the last page):

```rust
let mut mesh: PolygonMesh =
   obj::read(include_bytes!(concat!(env!("CARGO_MANIFEST_DIR"), "/output/sphere.obj")).as_slice())
      .unwrap();
println!("default shell condition: {:?}", mesh.shell_condition());
```

<details>
<summary>explanation</summary>

**Code breakdown**
<br>
  - `obj::read(...)`: loads the embedded `sphere.obj` into a `PolygonMesh`.
  - `mesh.shell_condition()`: inspects topology flags (irregular, oriented, closed).
  - `println!`: prints the detected condition for quick inspection.
  <br>

  <br>

**Conditions**
<br>

- **Irregular**: an edge has 3+ faces.
- **Regular**: each edge has at most two faces.
- **Oriented**: no edge appears twice with the same direction.
- **Closed**: every edge appears exactly twice (watertight manifold).

![condition](images/condition.png)

</details>

## Merge duplicate vertices

Remove seams created by duplicated coordinates:

```rust
// Merge duplicate vertices within 1e-3
mesh.put_together_same_attrs(1.0e-3);
println!("after merge: {:?}", mesh.shell_condition());
```

<details>
<summary>when to use</summary>

Importing OBJ/patch models, fixing seams, removing duplicate geometry.

</details>

## Add normals

Faceted (per-face) normals:

```rust
// Flat normals for a faceted look
mesh.add_naive_normals(true);
write_polygon_mesh(&mesh, "output/mirror-ball.obj");
```

<details>
<summary>when to use</summary>

Sharp mechanical parts, crisp reflections, debugging face orientation.

</details>

## Smooth normals, normalize normals

- Blend across angles for softer shading:
- Keep direction, fix length:

```rust
// Smooth normals for softer shading
mesh.add_smooth_normals(1.0, true); // ~57° crease angle
mesh.normalize_normals(); // keep normal lengths unit after any edits/imports
write_polygon_mesh(&mesh, "output/mirror-ball-with-smooth-normal.obj");


```

<details>
<summary>when to use <code>write_polygon_mesh</code></summary>

Spheres or organic shapes needing soft shading; lower the angle to preserve creases.

</details>

<details>
<summary>when to use <code>normalize_normals</code></summary>

After editing/importing normals to ensure they remain unit length without recomputing direction.

</details>



## Other cleanup passes

Truck’s filter module (`truck_meshalgo::filters`) also includes:

<details>
<summary><code>OptimizingFilter</code></summary>

- Drops **degenerate faces** (zero-area or repeated-vertex polygons).
- Removes **unused attributes** left over after edits or imports.
- Unifies **shared vertices** more aggressively than simple epsilon-based merging.

Useful after **imports**, **procedural generation**, or **tessellation passes** to reduce mesh size and fix broken topology.

</details>

<details>
<summary><code>StructuringFilter</code></summary>

- Reorganizes **vertex attributes** for tighter packing.
- Reorders **faces and indices** to improve cache-coherent traversal.
- Produces meshes that render faster and export more cleanly (OBJ/GLTF).

Useful before **export** or before sending the mesh into a **real-time renderer**.

</details>

<br>

## Subdivision and refinement

<details>
<summary> </summary>

- Use **Subdivision filters** to add geometric detail to coarse meshes.  
  (For example: smoothing a low-poly model before export or rendering.)
- Subdivision creates **refined faces and smoothed vertex flow**, improving shading quality.
- Always **apply subdivision after cleanup** (merging duplicates, removing degenerates, orienting) so the new mesh inherits **clean topology and correct normals**.

Useful for smooth surfaces, rounded objects, or preparing assets for high-fidelity viewers.

</details>


## Save and run

Use the existing `write_polygon_mesh` helper to export OBJ files, then:

```bash
cargo run
```

<br>

Inspect the results in an OBJ viewer (Preview, Blender, ParaView) to verify topology and normals.

![Sphere comparison](images/mirrorballs.gif)

<details>
<summary>Full example: filter a sphere OBJ (`examples/normals_filter_sphere.rs`)</summary>

```rust
use truck_meshalgo::prelude::*;
use truck_meshes::write_polygon_mesh;

fn main() {
    // Load the generated sphere OBJ from output/ embedded at compile time
    let mut mesh: PolygonMesh =
        obj::read(include_bytes!(concat!(env!("CARGO_MANIFEST_DIR"), "/output/sphere.obj")).as_slice())
            .unwrap();
    println!("default shell condition: {:?}", mesh.shell_condition());

    // Merge duplicate vertices within 1e-3
    mesh.put_together_same_attrs(1.0e-3);
    println!("after merge: {:?}", mesh.shell_condition());

    // Flat normals for a faceted look
    mesh.add_naive_normals(true);
    write_polygon_mesh(&mesh, "output/mirror-ball.obj");

    // Smooth normals for softer shading
    mesh.add_smooth_normals(1.0, true); // ~57° crease angle
    mesh.normalize_normals(); // keep normal lengths unit after any edits/imports
    write_polygon_mesh(&mesh, "output/mirror-ball-with-smooth-normal.obj");
}

```

</details>

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
├─ examples/
│  ├─ triangle.rs
│  ├─ square.rs
│  ├─ tetrahedron.rs
│  ├─ hexahedron.rs
│  ├─ octahedron.rs
│  ├─ dodecahedron.rs
│  ├─ icosahedron.rs
│  ├─ normals_icosahedron.rs
│  ├─ normals_sphere.rs
│  └─ normals_filter_sphere.rs
└─ output/          # exported OBJ files from examples
```
</details>
