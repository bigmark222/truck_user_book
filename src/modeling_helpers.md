# B-rep Helper Functions

Centralize OBJ/[STEP](https://en.wikipedia.org/wiki/ISO_10303-21) exports in `src/lib.rs` so shape modules (sibling files `cube.rs`, `torus.rs`, etc.) only model geometry.

**src/lib.rs** (helpers live here; shapes live as sibling files in `src/`)

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};
```

```rust
/// Export any B-rep (Solid or Shell) to STEP.
pub fn save_step_any<T, P>(brep: &T, path: P) -> std::io::Result<()>
where
    T: Compress,
    for<'a> StepModel<'a>: From<&'a T>,
    P: AsRef<std::path::Path>,
{
    let compressed = brep.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(path, display.to_string())
}
```

<details>
  <summary><strong>What these trait bounds mean</strong></summary>

  These bounds look scary, but each piece serves a simple purpose. Here's the breakdown:

  <details>
    <summary><strong>T: Compress</strong></summary>
    <p>
      <code>T</code> is a <a href="https://doc.rust-lang.org/rust-by-example/generics.html">generic type parameter.</a>  
      Saying <code>T: Compress</code> means: <br>
      <em>"Whatever T is, it must implement the Compress trait."</em><br><br>
      In Truck, both <code>Solid</code> and <code>Shell</code> implement <code>Compress</code>,  
      so this lets one function handle either type.
    </p>
  </details>

  <details>
    <summary><strong>for<'a> StepModel<'a>: From<&'a T></strong></summary>
    <p>
      This is a <em><a href="https://doc.rust-lang.org/nomicon/hrtb.html">higher-rank trait bound</a></em> (HRTB).  
      It means: <br>
      <em>"For every possible lifetime 'a, StepModel<'a> can be built from a reference to T."</em><br><br>
      In plain English:  
      <strong>“STEP export works for any &T you pass in.”</strong>
    </p>
  </details>

  <details>
    <summary><strong>P: AsRef&lt;Path&gt;</strong></summary>
    <p>
      This allows the path argument to be <code>&str</code>, <code>String</code>, <code>&Path</code>, or <code>PathBuf</code>.  
      It adds convenience—not a new feature.
    </p>
  </details>

</details>


<details>
  <summary>How <code>save_step_any</code> works</summary>

- <code>T: Compress</code> — lets the function work on any B-rep (Solid or Shell) because both support <code>compress()</code>

- <code>StepModel::from(&compressed)</code> — converts the compressed shape into STEP format

 - <code>CompleteStepDisplay</code> — adds the STEP file header and formatting.

- <code>to_string()</code> — turns the STEP data into text.

- <code>write(path, ...)</code> — saves that STEP text to disk.


  <p><strong>In plain English:</strong><br>
  The function takes any shape, cleans it up, converts it to STEP, adds the header, and writes the final STEP file to the path you give it.</p>
</details>
<br>

```rust
/// Triangulate any B-rep (Solid or Shell) and write an OBJ mesh.
pub fn save_obj(shape: &impl MeshedShape, path: &str) {
    let mesh = shape.triangulation(0.01).to_polygon();
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(&mesh, &mut obj).unwrap();
}
```

<details>
<summary>How <code>save_obj</code> works</summary>

1. `triangulation(0.01)` tessellates the analytic shape with a 0.01 chord tolerance, yielding a mesh that still carries topology.
2. `to_polygon()` drops adjacency/topology and keeps only polygon soup (positions, normals, UVs if present).
3. Create/truncate the OBJ file at `path`.
4. `obj::write` emits the mesh as a plain OBJ text file.

</details>

<br>
<br>

<details>
<summary>Directory tree</summary>

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
</details>

<details>
<summary>Complete code</summary>

**src/lib.rs**

```rust
use truck_modeling::prelude::*;
use truck_meshalgo::prelude::*;
use truck_stepio::{CompleteStepDisplay, StepModel};

/// Export any B-rep (Solid or Shell) to STEP.
pub fn save_step_any<T, P>(brep: &T, path: P) -> std::io::Result<()>
where
    T: Compress,
    for<'a> StepModel<'a>: From<&'a T>,
    P: AsRef<std::path::Path>,
{
    let compressed = brep.compress();
    let display = CompleteStepDisplay::new(
        StepModel::from(&compressed),
        Default::default(),
    );
    std::fs::write(path, display.to_string())
}

/// Triangulate any B-rep (Solid or Shell) and write an OBJ mesh.
pub fn save_obj(shape: &impl MeshedShape, path: &str) {
    let mesh = shape.triangulation(0.01).to_polygon();
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(&mesh, &mut obj).unwrap();
}
```
</details>
