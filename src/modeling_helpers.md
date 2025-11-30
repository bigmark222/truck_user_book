# B-rep Helper Functions

Centralize exports and reusable OBJ/STEP helpers in your `truck_brep` crate so shape files only worry about geometry.

## `src/lib.rs`

### Imports and module wiring

```rust
use std::{fs, io, path::Path};

use truck_meshalgo::prelude::*;
use truck_modeling::*;
use truck_stepio::out::{CompleteStepDisplay, StepModel};
use truck_topology::compress::{CompressedShell, CompressedSolid};
```

<details>
<summary>What this does</summary>
Brings in filesystem/IO utilities plus the Truck modeling, meshing, STEP output, and compression helpers that the rest of the files use.
</details>

### `StepCompress` trait

```rust
/// Helper to compress modeling shapes into STEP-compatible data.
pub trait StepCompress {
    type Compressed;
    fn compress_for_step(&self) -> Self::Compressed;
}
```

<details>
<summary>What this does</summary>
Defines a small adapter trait so both `Shell` and `Solid` expose a common `compress_for_step` method, hiding their different compressed types.
</details>

### `StepCompress` for `Shell`

```rust
impl StepCompress for Shell {
    type Compressed = CompressedShell<Point3, Curve, Surface>;
    fn compress_for_step(&self) -> Self::Compressed {
        self.compress()
    }
}
```

<details>
<summary>What this does</summary>
Implements `StepCompress` for shells by delegating to the kernel’s built-in `compress`, returning the STEP-ready shell representation.
</details>

### `StepCompress` for `Solid`

```rust
impl StepCompress for Solid {
    type Compressed = CompressedSolid<Point3, Curve, Surface>;
    fn compress_for_step(&self) -> Self::Compressed {
        self.compress()
    }
}
```

<details>
<summary>What this does</summary>
Implements `StepCompress` for solids, again forwarding to `compress` so solids can be exported with the same helper API.
</details>

### `save_step` helper

```rust
/// Export any B-rep (Solid or Shell) to STEP.
pub fn save_step<T, P>(brep: &T, path: P) -> io::Result<()>
where
    T: StepCompress,
    for<'a> StepModel<'a, Point3, Curve, Surface>: From<&'a T::Compressed>,
    P: AsRef<Path>,
{
    let path = path.as_ref();
    if let Some(parent) = path.parent() {
        fs::create_dir_all(parent)?;
    }
    let compressed = brep.compress_for_step();
    let display = CompleteStepDisplay::new(StepModel::from(&compressed), Default::default());
    fs::write(path, display.to_string())
}
```

<details>
<summary>What this does</summary>
Creates parent folders if needed, compresses the shape, wraps it in a STEP display with defaults, and writes the STEP text to disk.
</details>

### `save_obj` helper

```rust
/// Triangulate any B-rep (Solid or Shell) and write an OBJ mesh.
pub fn save_obj(shape: &impl MeshableShape, path: impl AsRef<Path>) -> io::Result<()> {
    let mesh = shape.triangulation(0.01).to_polygon();
    let path = path.as_ref();
    if let Some(parent) = path.parent() {
        fs::create_dir_all(parent)?;
    }
    let mut obj = fs::File::create(path)?;
    obj::write(&mesh, &mut obj).map_err(|err| io::Error::new(io::ErrorKind::Other, err))
}
```

<details>
<summary>What this does</summary>
Tessellates the shape with a 0.01 chord tolerance, ensures the output folder exists, then writes the resulting polygon mesh as an OBJ file.
</details>

<br>

<details>
<summary><strong>Target directory tree</strong></summary>

```
truck_brep/
├─ Cargo.toml      # add truck-modeling, truck-meshalgo, truck-stepio deps
├─ src/
│  ├─ lib.rs       # helpers + re-exports
│  ├─ ...          # shape modules (add torus.rs, cylinder.rs, bottle.rs, etc.)
├─ examples/
│  └─ ...          # one small example per shape
└─ output/         # generated OBJ/STEP files (created at runtime)
```
</details>

<details>
<summary><strong>Complete <code>src/lib.rs</code></strong></summary>

```rust
use std::{fs, io, path::Path};

use truck_meshalgo::prelude::*;
use truck_modeling::*;
use truck_stepio::out::{CompleteStepDisplay, StepModel};
use truck_topology::compress::{CompressedShell, CompressedSolid};

/// Helper to compress modeling shapes into STEP-compatible data.
pub trait StepCompress {
    type Compressed;
    fn compress_for_step(&self) -> Self::Compressed;
}

impl StepCompress for Shell {
    type Compressed = CompressedShell<Point3, Curve, Surface>;
    fn compress_for_step(&self) -> Self::Compressed { self.compress() }
}

impl StepCompress for Solid {
    type Compressed = CompressedSolid<Point3, Curve, Surface>;
    fn compress_for_step(&self) -> Self::Compressed { self.compress() }
}

/// Export any B-rep (Solid or Shell) to STEP.
pub fn save_step<T, P>(brep: &T, path: P) -> io::Result<()>
where
    T: StepCompress,
    for<'a> StepModel<'a, Point3, Curve, Surface>: From<&'a T::Compressed>,
    P: AsRef<Path>,
{
    let path = path.as_ref();
    if let Some(parent) = path.parent() {
        fs::create_dir_all(parent)?;
    }
    let compressed = brep.compress_for_step();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    fs::write(path, display.to_string())
}

/// Triangulate any B-rep (Solid or Shell) and write an OBJ mesh.
pub fn save_obj(shape: &impl MeshableShape, path: impl AsRef<Path>) -> io::Result<()> {
    let mesh = shape.triangulation(0.01).to_polygon();
    let path = path.as_ref();
    if let Some(parent) = path.parent() {
        fs::create_dir_all(parent)?;
    }
    let mut obj = fs::File::create(path)?;
    obj::write(&mesh, &mut obj).map_err(|err| io::Error::new(io::ErrorKind::Other, err))
}
```

</details>
