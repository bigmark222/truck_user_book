# Render a Mesh

Load a Truck-generated OBJ, set up a camera and a couple of lights, and render it with three-d.

Make sure your `Cargo.toml` includes these three-d crates:

```toml
three-d = { version = "0.18.2", features = ["window"] }
three-d-asset = { version = "0.9", features = ["obj"] }
```

## Imports and main

Start with the empty window example, then add the pieces below in this order:

- Keep the same `main` and `window` setup (no unwrap on `gl()`).
- After grabbing `context`, insert the `Camera::new_perspective` setup.
- Right after the camera, insert the directional and ambient lights.
- After the lights, load the OBJ into `cpu_mesh`, compute normals, and build `model`.
- Inside `render_loop`, keep `set_viewport(frame_input.viewport)` and render the model with both lights.

Full listing for reference:

```rust
use three_d::*;

fn main() {
    let window = Window::new(WindowSettings {
        title: "Truck mesh with three-d".into(),
        ..Default::default()
    })
    .unwrap();
    let context = window.gl();

    // Basic camera
    let mut camera = Camera::new_perspective(
        window.viewport(),
        vec3(5.0, 4.0, 5.0),
        vec3(0.0, 1.0, 0.0),
        vec3(0.0, 1.0, 0.0),
        degrees(55.0),
        0.1,
        50.0,
    );

    // White headlight + gentle fill
    let light_primary =
        DirectionalLight::new(&context, 1.0, Color::WHITE, &vec3(1.0, -1.0, -1.0)).unwrap();
    let light_fill = AmbientLight::new(&context, 0.2, Color::WHITE);

    // Load an OBJ exported from Truck (e.g., teapot.obj or cube.obj).
    let path = "output/mesh.obj";
    let mut cpu_mesh: three_d_asset::Mesh = three_d_asset::io::load(path)
        .expect("failed to read OBJ")
        .deserialize()
        .expect("no mesh in OBJ");
    cpu_mesh.compute_normals(); // ensure shading is smooth even if OBJ lacks normals

    let mut model = Model::new_with_material(
        &context,
        &cpu_mesh,
        PhysicalMaterial {
            albedo: Color::new_opaque(200, 200, 200),
            roughness: 0.35,
            metallic: 0.05,
            ..Default::default()
        },
    )
    .unwrap();

    window.render_loop(move |frame_input| {
        camera.set_viewport(frame_input.viewport);
        let screen = frame_input.screen();

        screen
            .render(
                ClearState::color_and_depth(0.06, 0.06, 0.08, 1.0, 1.0),
                |renderer| {
                    model.render(renderer, &camera, &[&light_primary, &light_fill]);
                    Ok(())
                },
            )
            .unwrap();

        FrameOutput::default()
    });
}
```

Replace `"output/mesh.obj"` with any OBJ generated in previous chapters. The normals step ensures shading looks correct even if the OBJ didn’t include them.

## Run

```bash
cargo run --bin viewer
```

A lit, shaded version of your mesh should appear immediately.
