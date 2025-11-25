# Reusable Viewer

Build a reusable viewer you can run against any OBJ coming out of the earlier chapters. It adds orbit controls, zoom, and optional drag-and-drop for quick inspection.

## Viewer structure

- Accepts a mesh path via CLI (`cargo run -- output/mesh.obj`).
- If the file is missing, shows a unit cube as a placeholder.
- Normalizes meshes to fit a 2-unit bounding sphere so you always start framed on the model.
- Orbit + scroll for navigation (from the previous section).
- Renders with a headlight + ambient light for consistent shading.

```rust
use three_d::*;

fn load_mesh(path: &str) -> three_d_asset::Mesh {
    three_d_asset::io::load(path)
        .and_then(|loaded| loaded.deserialize())
        .unwrap_or_else(|_| {
            eprintln!("{} not found, falling back to a unit cube", path);
            three_d_asset::Mesh::cube()
        })
}

fn normalize(mesh: &mut three_d_asset::Mesh) {
    if let Some(bbx) = mesh.compute_aabb() {
        let center = bbx.center();
        let radius = bbx.size().magnitude().max(1e-5) * 0.5;
        for p in mesh.positions_mut() {
            *p = (*p - center) / radius;
        }
    }
}

fn main() {
    let path = std::env::args().nth(1).unwrap_or_else(|| "output/mesh.obj".into());
    let mut cpu_mesh = load_mesh(&path);
    normalize(&mut cpu_mesh);
    cpu_mesh.compute_normals();

    // ... create window/camera/lights as before ...
    // let mut model = Model::new_with_material(&context, &cpu_mesh, PhysicalMaterial::default())?;
    // inside render_loop: orbit camera + model.render(...)
}
```

## Optional drag-and-drop

Threed exposes dropped file paths via `frame_input.dropped_file_paths`. If one is present, re-run `load_mesh -> normalize -> compute_normals -> Model::new_with_material` to swap in the new geometry. You do not need to manually manage GPU buffers—the Model constructor handles uploads for you.

## Run with auto-reload

Pair it with `cargo watch` to keep the viewer fresh as you regenerate meshes:

```bash
cargo watch -w output/mesh.obj -x 'run -- output/mesh.obj'
```

Now the viewer restarts whenever your modeling code replaces `output/mesh.obj`, and you can also drop other OBJs directly onto the window to swap models instantly.
