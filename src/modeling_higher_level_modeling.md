# Higher Level Modeling (Bottle)

Model a bottle using Truck’s lower-level B-rep APIs (inspired by the classic OCCT bottle tutorial). This walks through building parts, adding openings, and creating an inner cavity.

Reference: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter3/src/section3_3.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter3/src/section3_3.rs)

## Components

- Neck (cylinder shell)
- Body bottom and walls (arc + homotopy + sweep)
- Ceiling cutout (hole)
- Gluing neck to body
- Inner cavity (inverted inner shell)

## Helper: cylindrical shell

Returns a `Shell` (not `Solid`) so we can manipulate boundaries later.

```rust
use truck_modeling::prelude::*;
use truck_modeling::builder;

fn cylinder(bottom: f64, height: f64, radius: f64) -> Shell {
    let vertex = builder::vertex(Point3::new(0.0, bottom, radius));
    let circle = builder::rsweep(&vertex, Point3::origin(), Vector3::unit_y(), Rad(7.0));
    let disk = builder::try_attach_plane(&vec![circle]).unwrap();
    let solid = builder::tsweep(&disk, Vector3::new(0.0, height, 0.0));

    solid.into_boundaries().pop().unwrap() // extract shell
}
```

Minimal test:

```rust
fn bottle(_h: f64, _w: f64, _t: f64) -> Solid {
    Solid::new(vec![cylinder(-0.5, 1.0, 0.5)])
}
```

## Body shell (arcs + homotopy + sweep)

Use `circle_arc` to define a profile, `rotated` for the opposite side, `homotopy` to span a face, then `tsweep` upward.

```rust
fn body_shell(bottom: f64, height: f64, width: f64, thickness: f64) -> Shell {
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

Quick test:

```rust
fn bottle(h: f64, w: f64, t: f64) -> Solid {
    Solid::new(vec![body_shell(-h / 2.0, h, w, t)])
}
```

## Add a ceiling hole

Modify the top face by adding a circular boundary.

```rust
fn add_ceiling_hole(mut body: Shell, height: f64, thickness: f64) -> Shell {
    let ceiling = body.last_mut().unwrap();

    let circle = builder::rsweep(
        &builder::vertex(Point3::new(thickness / 4.0, height / 2.0, 0.0)),
        Point3::new(0.0, height / 2.0, 0.0),
        -Vector3::unit_y(),
        Rad(7.0),
    );

    ceiling.add_boundary(circle);
    body
}
```

## Glue body and neck

Attach the neck shell to the body shell by sharing the ceiling boundary.

```rust
fn glue_body_neck(body: &mut Shell, neck: Shell) {
    let body_ceiling = body.last_mut().unwrap();
    let wire = neck[0].boundaries()[0].clone();

    body_ceiling.add_boundary(wire);    // punch hole for neck
    body.extend(neck.into_iter().skip(1)); // add remaining neck faces
}

fn bottle(height: f64, width: f64, thickness: f64) -> Solid {
    let mut body = body_shell(-height / 2.0, height, width, thickness);
    let neck = cylinder(height / 2.0, height / 10.0, thickness / 4.0);

    glue_body_neck(&mut body, neck);
    Solid::new(vec![body])
}
```

## Add inner cavity (hollow bottle)

Create a slightly smaller inner shell, invert faces, and sew it into the outer shell.

```rust
fn bottle(height: f64, width: f64, thickness: f64) -> Solid {
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

    // Flip inner faces inward
    inner_body.face_iter_mut().for_each(|face| face.invert());

    // Remove inner ceiling and use its boundary as a hole in the outer ceiling
    let inner_ceiling = inner_body.pop().unwrap();
    let wire = inner_ceiling.into_boundaries().pop().unwrap();
    let ceiling = body.last_mut().unwrap();
    ceiling.add_boundary(wire);

    // Sew inner shell into outer shell
    body.extend(inner_body.into_iter());

    Solid::new(vec![body])
}
```

## Result

- Outer body + neck
- Cutout opening
- Boundary editing
- Hollow interior with correct face orientations

<details>
<summary>Directory tree (for this section)</summary>

```
truck_brep/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs
│  └─ bin/
│     └─ bottle.rs
└─ output/            # bottle.obj, bottle.step (or other names you choose)
```

</details>

<details>
<summary>Complete code (src/bin/bottle.rs)</summary>

```rust
use std::f64::consts::PI;
use truck_modeling::builder;
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

fn cylinder(bottom: f64, height: f64, radius: f64) -> Shell {
    let vertex = builder::vertex(Point3::new(0.0, bottom, radius));
    let circle = builder::rsweep(&vertex, Point3::origin(), Vector3::unit_y(), Rad(7.0));
    let disk = builder::try_attach_plane(&vec![circle]).unwrap();
    let solid = builder::tsweep(&disk, Vector3::new(0.0, height, 0.0));
    solid.into_boundaries().pop().unwrap()
}

fn body_shell(bottom: f64, height: f64, width: f64, thickness: f64) -> Shell {
    let v0 = builder::vertex(Point3::new(-width / 2.0, bottom, thickness / 4.0));
    let v1 = builder::vertex(Point3::new(width / 2.0, bottom, thickness / 4.0));
    let transit = Point3::new(0.0, bottom, thickness / 2.0);

    let arc0 = builder::circle_arc(&v0, &v1, transit);
    let arc1 = builder::rotated(&arc0, Point3::origin(), Vector3::unit_y(), Rad(PI));

    let face = builder::homotopy(&arc0, &arc1.inverse());
    let solid = builder::tsweep(&face, Vector3::new(0.0, height, 0.0));
    solid.into_boundaries().pop().unwrap()
}

fn add_ceiling_hole(mut body: Shell, height: f64, thickness: f64) -> Shell {
    let ceiling = body.last_mut().unwrap();
    let circle = builder::rsweep(
        &builder::vertex(Point3::new(thickness / 4.0, height / 2.0, 0.0)),
        Point3::new(0.0, height / 2.0, 0.0),
        -Vector3::unit_y(),
        Rad(7.0),
    );
    ceiling.add_boundary(circle);
    body
}

fn glue_body_neck(body: &mut Shell, neck: Shell) {
    let body_ceiling = body.last_mut().unwrap();
    let wire = neck[0].boundaries()[0].clone();
    body_ceiling.add_boundary(wire);
    body.extend(neck.into_iter().skip(1));
}

fn bottle(height: f64, width: f64, thickness: f64) -> Solid {
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

fn save_shape(solid: &Solid, name: &str) {
    let mesh_with_topology = solid.triangulation(0.01);
    let mesh = mesh_with_topology.to_polygon();
    let mut obj = std::fs::File::create(format!("output/{name}.obj")).unwrap();
    obj::write(&mesh, &mut obj).unwrap();

    let compressed = solid.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(format!("output/{name}.step"), display.to_string()).unwrap();
}

fn main() {
    let bottle = bottle(2.0, 1.0, 0.6);
    save_shape(&bottle, "bottle");
}
```

</details>
