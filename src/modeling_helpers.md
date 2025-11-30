# B-rep Helper Functions

Centralize exports and reusable OBJ/STEP helpers in your `truck_brep` crate so shape files only worry about geometry.

**src/lib.rs**

```rust
use std::{fs, io, path::Path};

use truck_meshalgo::prelude::*;
use truck_modeling::*;
use truck_stepio::out::{CompleteStepDisplay, StepModel};
use truck_topology::compress::{CompressedShell, CompressedSolid};
```

```rust
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
pub fn save_step_any<T, P>(brep: &T, path: P) -> io::Result<()>
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

/// Convenience wrapper matching the tutorial name.
pub fn save_step<T, P>(brep: &T, path: P) -> io::Result<()>
where
    T: StepCompress,
    for<'a> StepModel<'a, Point3, Curve, Surface>: From<&'a T::Compressed>,
    P: AsRef<Path>,
{
    save_step_any(brep, path)
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

- `StepCompress` unifies the slightly different compression types for `Shell` and `Solid`, so the STEP helpers can be generic without leaking topology details into every shape file.
- `save_step_any`/`save_step` create parent directories on demand, then write out formatted STEP text.
- `save_obj` tessellates with a small chord tolerance, ensures the output folder exists, then emits an OBJ mesh.
