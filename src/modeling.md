# Modeling

Shift from **polygon meshes** to precise **boundary representations (B-reps)**.  
Although both meshes and B-reps use vertices, edges, and faces, they **describe geometry in completely different ways.**

<br>

**Meshes** approximate shape using flat pieces:

- **Vertices:** points in 3D space
- **Edges:** straight line segments  
- **Faces:** flat polygons (usually triangles)

A **mesh** sphere, cylinder, or fillet is always a **faceted approximation** (represented using many small flat faces).

<br>

**B-reps** store the *exact* analytic geometry of a solid:

- **Vertices:** points in 3D space  
- **Edges:** mathematical curves (lines, arcs, splines, NURBS)  
- **Faces:** analytic surfaces (planes, cylinders, spheres, NURBS patches)  
- **Solids:** closed collections of faces forming true watertight bodies  

With **B-reps**, a circle is a true circle, a sphere is a perfect sphere, and fillets/blends are smooth by definition—not approximated by polygons.

This is the representation used by major CAD kernels:  
**Parasolid** (SolidWorks, Onshape, NX), **ACIS/ShapeManager** (Inventor), **CGM** (CATIA), and others.

<br>
<br>

<details>
  <summary><strong>Why use B-reps?</strong></summary>

  **B-reps (Boundary Representations)** are used in CAD because they preserve the *exact* geometry of a solid.  
  Unlike polygon meshes—which approximate surfaces with many small flat facets—B-reps retain the true analytic curves and surfaces that define engineered parts.

  **Advantages of B-reps:**

  - **Exact mathematical curves and surfaces**  
    Lines, arcs, circles, splines, NURBS, cylinders, spheres—no approximation.

  - **Dimension-driven modeling**  
    Features can be defined by precise measurements (radius, thickness, angle, sweep distance).

  - **Smooth surfaces by definition**  
    Fillets, blends, and revolved surfaces are perfectly smooth, not faceted.

  - **Export to neutral CAD formats** (STEP, IGES)  
    These formats require analytic geometry; meshes cannot represent them accurately.

  - **Clean conversion to meshes** for rendering or simulation  
    Tessellated meshes inherit exact topology from the B-rep, producing cleaner normals and fewer artifacts.

  Truck includes a complete B-rep system that you will use throughout Chapter 3.
</details>



## Project layout for Chapter 3

Keep your Chapter 2 mesh crate as-is, and add a sibling B-rep crate. Target workspace layout:

```
truck-workspace/
├─ truck_meshes/   # Chapter 2 mesh crate (unchanged)
└─ truck_brep/     # Chapter 3 B-rep crate you’ll create below
```

### Add a sibling crate dedicated to B-rep modeling

- Create a sibling crate named `truck_brep` in the parent directory that already contains `truck_meshes`:


```bash
# run from the parent dir that already has truck_meshes/
cargo new --lib truck_brep
mkdir -p truck_brep/examples truck_brep/output
```

- `src/lib.rs` holds the shared OBJ/STEP helpers and exports the per-shape modules.
- Per-shape modules (`cube.rs`, `torus.rs`, `cylinder.rs`, `bottle.rs`) sit directly in `src/` next to `lib.rs`.
- `examples/` holds runnable samples that call into `truck_brep::*`.
- `output/` collects OBJ/STEP exports so they do not clutter source control.

```
truck_brep/
├─ Cargo.toml      # add truck-modeling, truck-meshalgo, truck-stepio deps
├─ src/
│  ├─ lib.rs       # helpers + re-exports
│  └─              # add torus.rs, cylinder.rs, bottle.rs as you go
│ 
├─ examples/
│  └─              # calls truck_brep::cube(), etc.
└─ output/         # generated OBJ/STEP files
```

### Update `truck_brep/Cargo.toml`

Add the Truck crates (match the versions you used in `truck_meshes` so everything stays in sync):

```toml
[dependencies]
# modeling API
truck-modeling = "0.6.0"
# meshing modeled shape
truck-meshalgo = "0.4.0"
# output STEP
truck-stepio = "0.3.0"
# topology helpers (compression)
truck-topology = "0.6.0"

```

<br>

<details>
  <summary><strong>Tessellate B-reps to Meshes</strong></summary>

  To render or export, **B-rep solids must be tessellated** into polygon meshes.  
  Truck’s meshing layer [`truck-meshalgo`](https://github.com/ricosjp/truck/tree/master/truck-meshalgo) provides core [traits](https://doc.rust-lang.org/book/ch10-02-traits.html) for this process:

  - **MeshableShape**  
  Converts an analytic B-rep shape into a polygon mesh using default tessellation settings.

  - **RobustMeshableShape**  
  A more defensive version of `MeshableShape` that adds numerical tolerances and safety checks for difficult or complex shapes (CSG trees, tiny edges, near-degenerate surfaces).

  - **MeshedShape**  
  Represents the final tessellated result — a `PolygonMesh` paired with the metadata used during tessellation. 


  Truck also includes a tessellation demo, [`examples/tessellate-shape`](https://github.com/ricosjp/truck/blob/master/truck-meshalgo/examples/tessellate-shape.rs), which converts analytic shapes and full CSG trees into OBJ meshes.

  **Typical workflow:**

  1. Build or finish the analytic (B-rep) solid.  
  2. **Tessellate** it into a polygon mesh.  
  3. Run mesh filters (**normals**, **smoothing**, **optimization**).  
  4. Export or view the cleaned mesh in your preferred viewer.

  This ensures the final mesh accurately reflects the B-rep geometry while remaining efficient for rendering.
</details>


## What you will learn in Chapter 3

- **3.1 Cube**: build an exact cube; see how B-rep solids are assembled.
- **3.2 Torus**: work with rotational sweeps to create curved surfaces.
- **3.3 Cylinder**: combine rotational, planar, and translational sweeps for a solid.
- **3.4 Higher Level Modeling**: model with NURBS; build solids from exact curved surfaces instead of approximations.

By the end of this chapter, you’ll understand how Truck represents CAD-quality shapes—far more precise than meshes.

<details>
<summary>Reference: full truck_brep snapshot</summary>

Directory tree:

```
truck_brep/
├─ Cargo.toml      # add truck-modeling, truck-meshalgo, truck-stepio deps
├─ src/
│  ├─ lib.rs       # helpers + re-exports
│  └─              # add torus.rs, cylinder.rs, bottle.rs as you go
│ 
├─ examples/
│  └─              # calls truck_brep::cube(), etc.
└─ output/         # generated OBJ/STEP files
```


`Cargo.toml` dependencies:

```toml
[dependencies]
# modeling API
truck-modeling = "0.6.0"
# meshing modeled shape
truck-meshalgo = "0.4.0"
# output STEP
truck-stepio = "0.3.0"
# topology helpers (compression)
truck-topology = "0.6.0"

```
