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
```

### 1. Neck shell helper (`cylinder`)

```rust
/// Shell for the neck or inner neck.
pub fn cylinder(bottom: f64, height: f64, radius: f64) -> Shell {
    let vertex = builder::vertex(Point3::new(0.0, bottom, radius));
    let circle = builder::rsweep(&vertex, Point3::origin(), Vector3::unit_y(), Rad(7.0));
    let disk = builder::try_attach_plane(&vec![circle]).unwrap();
    let solid = builder::tsweep(&disk, Vector3::new(0.0, height, 0.0));
    solid.into_boundaries().pop().unwrap() // extract shell
}
```

<details>
<summary>How <code>cylinder()</code> works</summary>

1. Rotate a single vertex around the +Y axis to generate a circular wire at the chosen radius.
2. Seal that wire into a planar disk using `try_attach_plane`.
3. Sweep the disk upward along +Y to form a solid, then extract its outer shell.

</details>

### 2. Body shell helper (`body_shell`)

```rust
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
```

<details>
<summary>How <code>body_shell()</code> works</summary>

1. Construct two symmetric arcs across the body width (`arc0`, and `arc1` rotated 180°).
2. Loft between the arcs using `homotopy` to generate a smooth side surface.
3. Sweep that surface upward along +Y to create the body shell, then extract it.

</details>

### 3. Glue neck onto body (`glue_body_neck`)

```rust
/// Punch a hole in the ceiling and glue neck faces on top.
pub fn glue_body_neck(body: &mut Shell, neck: Shell) {
    let body_ceiling = body.last_mut().unwrap();
    let wire = neck[0].boundaries()[0].clone();

    // This boundary punch is the ceiling hole for the neck.
    body_ceiling.add_boundary(wire);       // punch hole for neck
    body.extend(neck.into_iter().skip(1)); // add remaining neck faces
}
```

<details>
<summary>How <code>glue_body_neck()</code> works</summary>

1. Select the top face of the body (`body_ceiling`) and the rim wire from the neck shell.
2. Insert the rim as an inner boundary to cut the neck opening in the ceiling face.
3. Attach the remaining neck faces to the body shell.

</details>

### 4. Assemble the full bottle (`bottle`)

```rust
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
```

<details>
<summary>How <code>bottle()</code> works</summary>

1. Construct the outer body and attach the neck shell.
2. Create a slightly smaller inner body and neck, attach them, then invert their faces.
3. Move the inner rim to the outer ceiling as a boundary hole and stitch in the inner shell.
4. Package the resulting shell hierarchy into a `Solid`.

</details>

## Expose the bottle module through lib.rs

**src/lib.rs**

```rust
pub mod shapes;
pub use shapes::{
    bottle, cube, cylinder, torus,
};
pub mod helpers;
pub use helpers::{save_obj, save_step_any};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub mod torus;
pub mod cylinder;
pub mod bottle;

pub use cube::cube;
pub use torus::torus;
pub use cylinder::cylinder;
pub use bottle::bottle;
```

## Example entry point

`examples/bottle.rs` just calls into the library:

```rust
fn main() {
    let bottle = truck_brep::bottle(2.0, 1.0, 0.6);
    truck_brep::save_obj(&bottle, "output/bottle.obj");
    truck_brep::save_step_any(&bottle, "output/bottle.step");
}
```

<details>
<summary>Directory tree (for this section)</summary>

```
truck_brep/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs              # re-exports helpers + shapes
│  ├─ helpers/
│  │  └─ mod.rs           # shared OBJ/STEP helpers
│  ├─ shapes/
│  │  ├─ mod.rs           # exports cube, torus, cylinder, bottle, ...
│  │  ├─ cube.rs
│  │  ├─ torus.rs
│  │  ├─ cylinder.rs
│  │  └─ bottle.rs        # this section
│  └─ examples/
│     ├─ cube.rs
│     ├─ torus.rs
│     ├─ cylinder.rs
│     └─ bottle.rs        # calls into lib
└─ output/                # cube.obj, torus.obj, cylinder.obj, bottle.obj, etc.
```

</details>

<details>
<summary>Complete code (lib + shapes + example)</summary>

**src/lib.rs**

```rust
pub mod shapes;
pub use shapes::{
    bottle, cube, cylinder, torus,
};
pub mod helpers;
pub use helpers::{save_obj, save_step_any};
```

**src/shapes/mod.rs**

```rust
pub mod cube;
pub mod torus;
pub mod cylinder;
pub mod bottle;

pub use cube::cube;
pub use torus::torus;
pub use cylinder::cylinder;
pub use bottle::bottle;
```

**src/shapes/bottle.rs**

```rust
use std::f64::consts::PI;
use truck_modeling::builder;
use truck_modeling::prelude::*;

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
```

**examples/bottle.rs**

```rust
fn main() {
    let bottle = truck_brep::bottle(2.0, 1.0, 0.6);
    truck_brep::save_obj(&bottle, "output/bottle.obj");
    truck_brep::save_step_any(&bottle, "output/bottle.step");
}
```

</details>
