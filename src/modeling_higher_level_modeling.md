# Higher Level Modeling (Bottle)

Model a bottle using Truck’s lower-level B-rep APIs (inspired by the classic OCCT bottle tutorial), keeping geometry and exports in the library just like the cube/torus/cylinder pages.

## Components

- Neck (cylindrical shell)
- Body bottom and walls (arc + homotopy + sweep)
- Ceiling cutout (hole)
- Gluing neck to body
- Inner cavity (inverted inner shell)

## Bottle in `src/shapes/bottle.rs`

Build everything inside the shapes module; binaries stay tiny and reusable.

```rust
use std::f64::consts::PI;
use truck_modeling::builder;
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

/// Shell for the neck or inner neck.
pub fn cylinder(bottom: f64, height: f64, radius: f64) -> Shell {
    let vertex = builder::vertex(Point3::new(0.0, bottom, radius));
    let circle = builder::rsweep(&vertex, Point3::origin(), Vector3::unit_y(), Rad(7.0));
    let disk = builder::try_attach_plane(&vec![circle]).unwrap();
    let solid = builder::tsweep(&disk, Vector3::new(0.0, height, 0.0));
    solid.into_boundaries().pop().unwrap() // extract shell
}

/// Outer/inner body shell built from arcs, homotopy, and sweep.
pub fn body_shell(bottom: f64, height: f64, width: f64, thickness: f64) -> Shell {
    let v0 = builder::vertex(Point3::new(-width / 2.0, bottom, thickness / 4.0));
    let v1 = builder::vertex(Point3::new(width / 2.0, bottom, thickness / 4.0));
    let transit = Point3::new(0.0, bottom, thickness / 2.0);

    let arc0 = builder::circle_arc(&v0, &v1, transit);
    let arc1 = builder::rotated(&arc0, Point3::origin(), Vector3::unit_y(), Rad(PI));

    let face = builder::homotopy(&arc0, &arc1.inverse());
    let solid = builder::tsweep(&face, Vector3::new(0.0, height, 0.0));
    solid.into_boundaries().pop().unwrap()
}

/// Punch a hole in the ceiling and glue neck faces on top.
pub fn glue_body_neck(body: &mut Shell, neck: Shell) {
    let body_ceiling = body.last_mut().unwrap();
    let wire = neck[0].boundaries()[0].clone();

    // This boundary punch is the ceiling hole for the neck.
    body_ceiling.add_boundary(wire);       // punch hole for neck
    body.extend(neck.into_iter().skip(1)); // add remaining neck faces
}

/// Hollow bottle with inner cavity and neck.
pub fn bottle(height: f64, width: f64, thickness: f64) -> Solid {
    let mut body = body_shell(-height / 2.0, height, width, thickness);
    let neck = cylinder(height / 2.0, height / 10.0, thickness / 4.0);
    glue_body_neck(&mut body, neck);

    let eps = height / 50.0;

    // Inner shell (shrunk and inset)
    let mut inner_body = body_shell(
        -height / 2.0 + eps,
        height - 2.0 * eps,
        width - 2.0 * eps,
        thickness - 2.0 * eps,
    );
    let inner_neck = cylinder(
        height / 2.0 - eps,
        height / 10.0 + eps,
        thickness / 4.0 - eps,
    );
    glue_body_neck(&mut inner_body, inner_neck);

    // Flip inner faces inward and sew into outer shell
    inner_body.face_iter_mut().for_each(|face| face.invert());
    let inner_ceiling = inner_body.pop().unwrap();
    let wire = inner_ceiling.into_boundaries().pop().unwrap();
    let ceiling = body.last_mut().unwrap();
    ceiling.add_boundary(wire);
    body.extend(inner_body.into_iter());

    Solid::new(vec![body])
}

/// Triangulate and write an OBJ mesh.
pub fn save_bottle_obj(solid: &Solid, path: &str) {
    let mesh_with_topology = solid.triangulation(0.01);
    let mesh = mesh_with_topology.to_polygon();
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(&mesh, &mut obj).unwrap();
}

/// Export the exact B-rep to STEP.
pub fn save_bottle_step(solid: &Solid, path: &str) {
    let compressed = solid.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(path, display.to_string()).unwrap();
}
```

Bottle steps:

1. Build outer body shell from arcs + homotopy + sweep.
2. Build a neck shell and glue it to the body ceiling (adds a hole).
3. Build a slightly smaller inner shell, invert faces, and sew it in.
4. Export with OBJ/STEP helpers.

## Expose the bottle module through lib.rs

**src/lib.rs**

```rust
pub mod shapes;
pub use shapes::{
    cube, save_obj, save_step,
    torus, save_torus_obj, save_torus_step,
    cylinder, save_cylinder_obj, save_cylinder_step,
    bottle, save_bottle_obj, save_bottle_step,
};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub mod torus;
pub mod cylinder;
pub mod bottle;

pub use cube::{cube, save_obj, save_step};
pub use torus::{torus, save_torus_obj, save_torus_step};
pub use cylinder::{cylinder, save_cylinder_obj, save_cylinder_step};
pub use bottle::{bottle, save_bottle_obj, save_bottle_step};
```

## Main entry point (bin)

`src/bin/bottle.rs` just calls into the library:

```rust
fn main() {
    let bottle = truck_brep::bottle(2.0, 1.0, 0.6);
    truck_brep::save_bottle_obj(&bottle, "output/bottle.obj");
    truck_brep::save_bottle_step(&bottle, "output/bottle.step");
}
```

<details>
<summary>Directory tree (for this section)</summary>

```
truck_brep/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs              # re-exports shapes
│  ├─ shapes/
│  │  ├─ mod.rs           # exports cube, torus, cylinder, bottle, ...
│  │  ├─ cube.rs
│  │  ├─ torus.rs
│  │  ├─ cylinder.rs
│  │  └─ bottle.rs        # this section
│  └─ bin/
│     ├─ cube.rs
│     ├─ torus.rs
│     ├─ cylinder.rs
│     └─ bottle.rs        # calls into lib
└─ output/                # cube.obj, torus.obj, cylinder.obj, bottle.obj, etc.
```

</details>

<details>
<summary>Complete code (lib + shapes + bin)</summary>

**src/lib.rs**

```rust
pub mod shapes;
pub use shapes::{
    cube, save_obj, save_step,
    torus, save_torus_obj, save_torus_step,
    cylinder, save_cylinder_obj, save_cylinder_step,
    bottle, save_bottle_obj, save_bottle_step,
};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub mod torus;
pub mod cylinder;
pub mod bottle;

pub use cube::{cube, save_obj, save_step};
pub use torus::{torus, save_torus_obj, save_torus_step};
pub use cylinder::{cylinder, save_cylinder_obj, save_cylinder_step};
pub use bottle::{bottle, save_bottle_obj, save_bottle_step};
```

**src/shapes/bottle.rs**

```rust
use std::f64::consts::PI;
use truck_modeling::builder;
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

pub fn cylinder(bottom: f64, height: f64, radius: f64) -> Shell {
    let vertex = builder::vertex(Point3::new(0.0, bottom, radius));
    let circle = builder::rsweep(&vertex, Point3::origin(), Vector3::unit_y(), Rad(7.0));
    let disk = builder::try_attach_plane(&vec![circle]).unwrap();
    let solid = builder::tsweep(&disk, Vector3::new(0.0, height, 0.0));
    solid.into_boundaries().pop().unwrap()
}

pub fn body_shell(bottom: f64, height: f64, width: f64, thickness: f64) -> Shell {
    let v0 = builder::vertex(Point3::new(-width / 2.0, bottom, thickness / 4.0));
    let v1 = builder::vertex(Point3::new(width / 2.0, bottom, thickness / 4.0));
    let transit = Point3::new(0.0, bottom, thickness / 2.0);

    let arc0 = builder::circle_arc(&v0, &v1, transit);
    let arc1 = builder::rotated(&arc0, Point3::origin(), Vector3::unit_y(), Rad(PI));

    let face = builder::homotopy(&arc0, &arc1.inverse());
    let solid = builder::tsweep(&face, Vector3::new(0.0, height, 0.0));
    solid.into_boundaries().pop().unwrap()
}

pub fn glue_body_neck(body: &mut Shell, neck: Shell) {
    let body_ceiling = body.last_mut().unwrap();
    let wire = neck[0].boundaries()[0].clone();

    // This boundary punch is the ceiling hole for the neck.
    body_ceiling.add_boundary(wire);
    body.extend(neck.into_iter().skip(1));
}

pub fn bottle(height: f64, width: f64, thickness: f64) -> Solid {
    let mut body = body_shell(-height / 2.0, height, width, thickness);
    let neck = cylinder(height / 2.0, height / 10.0, thickness / 4.0);
    glue_body_neck(&mut body, neck);

    let eps = height / 50.0;
    let mut inner_body = body_shell(
        -height / 2.0 + eps,
        height - 2.0 * eps,
        width - 2.0 * eps,
        thickness - 2.0 * eps,
    );
    let inner_neck = cylinder(
        height / 2.0 - eps,
        height / 10.0 + eps,
        thickness / 4.0 - eps,
    );
    glue_body_neck(&mut inner_body, inner_neck);

    inner_body.face_iter_mut().for_each(|face| face.invert());
    let inner_ceiling = inner_body.pop().unwrap();
    let wire = inner_ceiling.into_boundaries().pop().unwrap();
    let ceiling = body.last_mut().unwrap();
    ceiling.add_boundary(wire);
    body.extend(inner_body.into_iter());

    Solid::new(vec![body])
}

pub fn save_bottle_obj(solid: &Solid, path: &str) {
    let mesh_with_topology = solid.triangulation(0.01);
    let mesh = mesh_with_topology.to_polygon();
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(&mesh, &mut obj).unwrap();
}

pub fn save_bottle_step(solid: &Solid, path: &str) {
    let compressed = solid.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(path, display.to_string()).unwrap();
}
```

**src/bin/bottle.rs**

```rust
fn main() {
    let bottle = truck_brep::bottle(2.0, 1.0, 0.6);
    truck_brep::save_bottle_obj(&bottle, "output/bottle.obj");
    truck_brep::save_bottle_step(&bottle, "output/bottle.step");
}
```

</details>
