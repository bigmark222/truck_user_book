# Icosahedron

An icosahedron is the [dual](https://en.wikipedia.org/wiki/Dual_polyhedron) of a dodecahedron: each dodecahedron face center becomes a vertex; the three centers around each original dodecahedron vertex become one triangle.

Steps:
- Find every dodecahedron face center.
- [Normalize](https://en.wikipedia.org/wiki/Normal_(geometry)) those centers to the [unit sphere](https://en.wikipedia.org/wiki/Unit_sphere) → 20 vertices.
- Group the three centers around each original dodecahedron vertex (corner) → 20 faces.

![Icosahedron from dodecahedron illustration](images/icosa_from_dodeca.png)

##### The yellow icosahedron appears if you remove the orange reference dodecahedron.

## Add the icosahedron module

`src/lib.rs` additions:

```rust
use truck_meshalgo::prelude::*;

// ...keep earlier functions through dodecahedron...
pub mod icosahedron; // add this
pub use icosahedron::icosahedron; // add this
```

## Construct Main Function

`src/icosahedron.rs`:

```rust
use truck_meshalgo::prelude::*;

/// Icosahedron via dual of a dodecahedron.
pub fn icosahedron() -> PolygonMesh {

    //PLACE STEPS 1-5 HERE

}
```

#### Step 1: Start from a dodecahedron

```rust
    let dodeca: PolygonMesh = crate::dodecahedron();
    let d_positions = dodeca.positions();
```

#### Step 2: Build vertex positions from face centroids

```rust
    let positions: Vec<Point3> = dodeca
        .face_iter()
        .map(|face| {
            let centroid = face
                .iter()
                .map(|v| d_positions[v.pos].to_vec())
                .sum::<Vector3>();
            Point3::from_vec(centroid.normalize())
        })
        .collect();
```

<details>
<summary>How this block builds icosahedron vertices</summary>

<ul>
  <li><code>dodeca.face_iter()</code> iterates over each pentagonal face of the source dodecahedron.</li>
  <li>For each face we grab its five vertex positions from <code>d_positions</code>, convert them to vectors, and sum them to get the face <a href="https://en.wikipedia.org/wiki/Centroid">centroid</a> vector. </li>
  <li><code>centroid.normalize()</code> projects that centroid direction onto the unit sphere so every new vertex sits on radius&nbsp;1.</li>
  <li><code>Point3::from_vec(...)</code> turns the normalized direction back into a point; collecting the results gives the 20 icosahedron vertex positions.</li>
</ul>
</details>

#### Step 3: Build faces by collecting touching centroids

```rust
    let mut faces: Faces = (0..20)
        .map(|i| {
            dodeca
                .face_iter()
                .enumerate()
                .filter(|(_, f)| f.contains(&i.into()))
                .map(|(idx, _)| idx)
                .collect::<Vec<usize>>()
        })
        .collect();
```

<details>
<summary>How this block builds icosahedron faces</summary>

<ul>
  <li>We iterate over each of the 20 original dodecahedron vertices (<code>0..20</code>), because every dodeca vertex becomes one icosahedron face.</li>
  <li>For a given dodeca vertex <code>i</code>, we scan all dodeca faces with <code>face_iter().enumerate()</code> and pick the ones that contain that vertex (<code>f.contains(&i.into())</code>).</li>
  <li>The indices of those touching faces are collected into a <code>Vec&lt;usize&gt;</code>; these indices correspond to the centroids computed earlier (which are now icosahedron vertices).</li>
  <li>Gathering all 20 such lists yields the 20 triangular faces of the icosahedron.</li>
</ul>
</details>

#### Step 4: Fix winding so normals point outward

```rust
    faces.face_iter_mut().for_each(|face| {
        let p: Vec<Point3> = face.iter().map(|v| positions[v.pos]).collect();
        let center = p[0].to_vec() + p[1].to_vec() + p[2].to_vec();
        let normal = (p[1] - p[0]).cross(p[2] - p[0]).normalize();
        if center.dot(normal) < 0.0 {
            face.swap(0, 1);
        }
    });
```

<details>
<summary>How this block orients face winding</summary>

<ul>
  <li>For each face we fetch its three vertex positions (<code>p</code>).</li>
  <li>We sum the three position vectors to get a rough face center direction (<code>center</code>); no division needed because only direction matters.</li>
  <li>We compute the face normal via the <a href="https://en.wikipedia.org/wiki/Cross_product">cross product</a> of two edges and normalize it.</li>
  <li>If the normal points inward (<code>center.dot(normal) &lt; 0</code>), we swap two vertices to flip the winding so the normal points outward.</li>
</ul>
</details>

#### Step 5: Construct the mesh

```rust
    PolygonMesh::new(
        StandardAttributes {
            positions,
            ..Default::default()
        },
        faces,
    )
```

## Export the icosahedron

Add `examples/icosahedron.rs`:

```rust
use truck_meshalgo::prelude::NormalFilters;

fn main() {
    let mut mesh = truck_meshes::icosahedron();
    mesh.add_naive_normals(true); // optional, for shading
    truck_meshes::write_polygon_mesh(&mesh, "output/icosahedron.obj");
}
```

Run it:

```bash
cargo run --example icosahedron
```

<details>
<summary>File tree after this step</summary>

```
truck_meshes/
├─ Cargo.toml
├─ src/
│  ├─ lib.rs
│  ├─ triangle.rs
│  ├─ square.rs
│  ├─ tetrahedron.rs
│  ├─ hexahedron.rs
│  ├─ octahedron.rs
│  ├─ dodecahedron.rs
│  └─ icosahedron.rs
├─ examples/
│  ├─ triangle.rs
│  ├─ square.rs
│  ├─ tetrahedron.rs
│  ├─ hexahedron.rs
│  ├─ octahedron.rs
│  ├─ dodecahedron.rs
│  └─ icosahedron.rs
└─ output/          # exported OBJ files (e.g., output/icosahedron.obj)
```

</details>

<details>
<summary>Full code:</summary>

`src/lib.rs`:

```rust
use truck_meshalgo::prelude::*;

pub fn write_polygon_mesh(mesh: &PolygonMesh, path: &str) {
    let mut obj = std::fs::File::create(path).unwrap();
    obj::write(mesh, &mut obj).unwrap();
}

pub mod triangle;
pub use triangle::triangle;

pub mod square;
pub use square::square;

pub mod tetrahedron;
pub use tetrahedron::tetrahedron;

pub mod hexahedron;
pub use hexahedron::hexahedron;

pub mod octahedron;
pub use octahedron::octahedron;

pub mod dodecahedron;
pub use dodecahedron::dodecahedron;

pub mod icosahedron;
pub use icosahedron::icosahedron;
```

`src/icosahedron.rs`:

```rust
use truck_meshalgo::prelude::*;

pub fn icosahedron() -> PolygonMesh {
    let dodeca: PolygonMesh = crate::dodecahedron();
    let d_positions = dodeca.positions();

    let positions: Vec<Point3> = dodeca
        .face_iter()
        .map(|face| {
            let centroid = face
                .iter()
                .map(|v| d_positions[v.pos].to_vec())
                .sum::<Vector3>();
            Point3::from_vec(centroid.normalize())
        })
        .collect();

    let mut faces: Faces = (0..20)
        .map(|i| {
            dodeca
                .face_iter()
                .enumerate()
                .filter(|(_, f)| f.contains(&i.into()))
                .map(|(idx, _)| idx)
                .collect::<Vec<usize>>()
        })
        .collect();

    faces.face_iter_mut().for_each(|face| {
        let p: Vec<Point3> = face.iter().map(|v| positions[v.pos]).collect();
        let center = p[0].to_vec() + p[1].to_vec() + p[2].to_vec();
        let normal = (p[1] - p[0]).cross(p[2] - p[0]).normalize();
        if center.dot(normal) < 0.0 {
            face.swap(0, 1);
        }
    });

    PolygonMesh::new(
        StandardAttributes {
            positions,
            ..Default::default()
        },
        faces,
    )
}
```

`examples/icosahedron.rs`:

```rust
use truck_meshalgo::prelude::NormalFilters;

fn main() {
    let mut mesh = truck_meshes::icosahedron();
    mesh.add_naive_normals(true); // optional, for shading
    truck_meshes::write_polygon_mesh(&mesh, "output/icosahedron.obj");
}

```

</details>
