# Modeling

Shift from polygon meshes to precise boundary representations (B-reps). A B-rep describes a solid by its boundaries:

- Vertices: points in 3D space
- Edges: curves connecting vertices
- Faces: surfaces bounded by edges
- Solids: closed collections of faces

Unlike triangle meshes (flat approximations), B-reps model exact geometry: true circles, exact arcs, mathematically perfect spheres, NURBS curves/surfaces, smooth fillets and blends. This is the approach used by CAD kernels and systems—Parasolid (SolidWorks, Onshape, NX), ACIS/ShapeManager (Inventor), CGM (CATIA), etc.

## Why use B-reps?

- Preserve exact mathematical curves
- Dimension-driven modeling
- Smooth surfaces (not faceted)
- Export to neutral CAD formats (STEP, IGES)
- Clean conversion to meshes for rendering or simulation

Truck includes a complete B-rep system you’ll use in the next sections.

## Project layout for Chapter 3

Keep your Chapter 2 mesh crate as-is:

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
│  ├─ shapes/
│  │  ├─ triangle.rs
│  │  ├─ square.rs
│  │  ├─ tetrahedron.rs
│  │  ├─ hexahedron.rs
│  │  ├─ octahedron.rs
│  │  ├─ dodecahedron.rs
│  │  └─ icosahedron.rs
│  └─ normals/
│     ├─ filter_sphere.rs
│     ├─ icosahedron.rs
│     └─ sphere.rs
└─ output/          # optional: generated OBJ files from examples
```

Create the mesh output folder once so example OBJ files land in a predictable place:

```bash
mkdir -p truck_meshes/output
```

Add a sibling crate dedicated to B-rep modeling so analytic solids and exports stay isolated. Name it `truck_brep` (create it from the parent directory that already contains `truck_meshes`):

```bash
# run from the parent dir that already has truck_meshes/
cargo new --lib truck_brep
mkdir -p truck_brep/src/shapes truck_brep/src/bin truck_brep/output
```

- `src/shapes/` holds B-rep shape modules (cube, torus, cylinder, bottle).
- `src/bin/` holds runnable samples that call into `truck_brep::shapes::*`.
- `output/` collects OBJ/STEP exports so they do not clutter source control.
- In `src/lib.rs` add `pub mod shapes;` and re-export any helpers you want at the crate root.

```
truck_brep/
├─ Cargo.toml   # add truck-modeling, truck-meshalgo, truck-stepio deps
├─ src/
│  ├─ lib.rs       # re-exports shapes
│  ├─ shapes/
│  │  ├─ mod.rs
│  │  └─ cube.rs   # add torus.rs, cylinder.rs, bottle.rs as you go
│  └─ bin/
│     └─ cube.rs   # calls truck_brep::cube(), etc.
└─ output/         # generated OBJ/STEP files
```

## Tessellate B-reps to meshes

To render or export, B-rep solids are tessellated into polygon meshes. Truck’s meshing layer (`truck-meshalgo`) provides traits like `MeshableShape`, `RobustMeshableShape`, and `MeshedShape` (see `src/tessellation/mod.rs` in the crate) plus a sample driver `examples/tessellate-shape` that turns analytic shapes and CSG trees into OBJ meshes. You’ll typically tessellate once you finish a solid, then pass the mesh through filters (normals, optimization) before viewing it.

## What you will learn in Chapter 3

- **3.1 Cube**: build an exact cube; see how B-rep solids are assembled.
- **3.2 Torus**: work with rotational sweeps to create curved surfaces.
- **3.3 Cylinder**: combine rotational, planar, and translational sweeps for a solid.
- **3.4 Higher Level Modeling**: model with NURBS; build solids from exact curved surfaces instead of approximations.

By the end of this chapter, you’ll understand how Truck represents CAD-quality shapes—far more precise than meshes.
