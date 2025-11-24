# Modeling

Shift from polygon meshes to precise boundary representations (B-reps). A B-rep describes a solid by its boundaries:

- Vertices: points in 3D space
- Edges: curves connecting vertices
- Faces: surfaces bounded by edges
- Solids: closed collections of faces

Unlike triangle meshes (flat approximations), B-reps model exact geometry: true circles, exact arcs, mathematically perfect spheres, NURBS curves/surfaces, smooth fillets and blends. This is the approach used by CAD systems (SolidWorks, CATIA, Inventor, Parasolid, etc.).

## Why use B-reps?

- Preserve exact mathematical curves
- Dimension-driven modeling
- Smooth surfaces (not faceted)
- Export to neutral CAD formats (STEP, IGES)
- Clean conversion to meshes for rendering or simulation

Truck includes a complete B-rep system you’ll use in the next sections.

## Tessellate B-reps to meshes

To render or export, B-rep solids are tessellated into polygon meshes. Truck’s meshing layer (`truck-meshalgo`) provides traits like `MeshableShape`, `RobustMeshableShape`, and `MeshedShape` (see `src/tessellation/mod.rs` in the crate) plus a sample driver `examples/tessellate-shape` that turns analytic shapes and CSG trees into OBJ meshes. You’ll typically tessellate once you finish a solid, then pass the mesh through filters (normals, optimization) before viewing it.

## What you will learn in Chapter 3

- **3.1 Cube**: build an exact cube; see how B-rep solids are assembled.
- **3.2 Torus**: work with rotational sweeps to create curved surfaces.
- **3.3 Cylinder**: combine rotational, planar, and translational sweeps for a solid.
- **3.4 Higher Level Modeling**: model with NURBS; build solids from exact curved surfaces instead of approximations.

By the end of this chapter, you’ll understand how Truck represents CAD-quality shapes—far more precise than meshes.
