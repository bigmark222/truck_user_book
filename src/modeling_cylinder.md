# Cylinder

## Build a cylinder with `rsweep`, `try_attach_plane`, and `tsweep`

<details>
  <summary><strong>What happens at each step</strong></summary>

  <ul>
    <li><strong>Rotational sweep (`rsweep`):</strong> spin a vertex to make a circular wire.</li>
    <li><strong>Plane attachment:</strong> cap the wire into a disk with <code>try_attach_plane</code>.</li>
    <li><strong>Translational sweep (`tsweep`):</strong> extrude the disk to form the solid.</li>
  </ul>
</details>

### `src/cylinder.rs`

```rust
use truck_modeling::*;
```

```rust
pub fn torus() -> Solid {
  // STEPS 1-3 GO HERE

}
```

#### 1. Place a vertex on the base circle at `(0, 0, -1)`.

```rust
  let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, -1.0));
```

<details>
<summary>visual</summary>

```
          y
          │
          │
   ●      │   (start at z = -1)
          │
          └──── x
         /
        z
```

</details>

#### 2. Spin the vertex around +Z to form a circular wire.

```rust
  let circle: Wire = builder::rsweep(
      &vertex,
      Point3::new(0.0, 1.0, -1.0), // point on rotation axis
      Vector3::unit_z(),          // axis direction
      Rad(7.0),                   // > 2π ensures closure
  );
```

<details>
<summary>visual</summary>

```
Axis: +Z through (0, 1, -1)

     ●---●---●   (circle in XY plane at z = -1)
      \  |  /
       \ | /
         +
        axis
```

</details>

#### 3. Cap the wire into a planar disk.

```rust
  let disk: Face =
      builder::try_attach_plane(&vec![circle]).expect("cannot attach plane");
```

<details>
<summary>visual</summary>

```
The circular wire is filled to a disk (face) if the wire is closed and planar.

     +------+
    /        \
   |    ●     |
    \        /
     +------+
```

</details>

#### 4. Extrude the disk 2 units along +Z to make the solid.

```rust
  builder::tsweep(&disk, 2.0 * Vector3::unit_z())
```

<details>
<summary>visual</summary>

```
Sweeping the disk upward (+Z) forms the cylinder volume:

     ●───────●
    /|       /|
   ●───────●  |
   | |      | |
   | ●──────|-●
   |/       |/
   ●───────●
```

</details>

## Update `src/lib.rs` to expose `cylinder`

```rust
pub mod cylinder;
pub use cylinder::cylinder;
```

Keep helpers in lib.rs and cylinder in `src/cylinder.rs`.

## Example entry point

`examples/cylinder.rs` stays tiny and calls into the library:

```rust
fn main() {
    let cylinder = truck_brep::cylinder();
    truck_brep::save_obj(&cylinder, "output/cylinder.obj").unwrap();
    truck_brep::save_step(&cylinder, "output/cylinder.step").unwrap();
}
```

<br>

<details>
<summary>Directory tree (for this section)</summary>

```
truck_brep/
├─ src/
│  ├─ lib.rs      # helpers + re-exports
│  ├─ cube.rs
│  ├─ torus.rs
│  └─ cylinder.rs # this section
├─ examples/
│  ├─ cube.rs
│  ├─ torus.rs
│  └─ cylinder.rs
└─ output/        # created at runtime
```

</details>

<details>
<summary>Complete code</summary>

**src/cylinder.rs**

```rust
use truck_modeling::*;

pub fn cylinder() -> Solid {
    let vertex: Vertex = builder::vertex(Point3::new(0.0, 0.0, -1.0));
    let circle: Wire = builder::rsweep(
        &vertex,
        Point3::new(0.0, 1.0, -1.0),
        Vector3::unit_z(),
        Rad(7.0),
    );
    let disk: Face = builder::try_attach_plane(&vec![circle]).expect("cannot attach plane");
    builder::tsweep(&disk, 2.0 * Vector3::unit_z())
}
```

**examples/cylinder.rs**

```rust
fn main() {
    let cylinder = truck_brep::cylinder();
    truck_brep::save_obj(&cylinder, "output/cylinder.obj").unwrap();
    truck_brep::save_step(&cylinder, "output/cylinder.step").unwrap();
}
```

</details>
