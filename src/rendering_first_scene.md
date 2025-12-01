# Render a Mesh

Load a Truck-generated OBJ, set up a camera and a couple of lights, and render it with Bevy.

Make sure your `Cargo.toml` includes Bevy (defaults keep the OBJ loader on):

```toml
[dependencies]
bevy = { version = "0.13", features = ["file_watcher"] }
```

> Put your OBJ files under `assets/` so Bevy’s `AssetServer` can find them (e.g., `assets/output/mesh.obj`).

## Imports and main

Start from the empty window example and add:

- A 3D camera pointing at the origin.
- A directional light plus some ambient light.
- A PBR mesh loaded from your Truck-generated OBJ.

Full listing for reference:

```rust
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(WindowPlugin {
            primary_window: Some(Window {
                title: "Truck mesh with Bevy".into(),
                ..Default::default()
            }),
            ..Default::default()
        }))
        .add_systems(Startup, setup)
        .run();
}

fn setup(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    commands.insert_resource(ClearColor(Color::srgb(0.06, 0.06, 0.08)));
    commands.insert_resource(AmbientLight {
        color: Color::WHITE,
        brightness: 0.2,
    });

    // Basic camera
    commands.spawn(Camera3dBundle {
        transform: Transform::from_xyz(5.0, 4.0, 5.0)
            .looking_at(Vec3::new(0.0, 1.0, 0.0), Vec3::Y),
        ..Default::default()
    });

    // White headlight + gentle fill
    commands.spawn(DirectionalLightBundle {
        directional_light: DirectionalLight {
            shadows_enabled: true,
            illuminance: 8_000.0,
            ..Default::default()
        },
        transform: Transform::from_xyz(1.0, -1.0, -1.0).looking_at(Vec3::ZERO, Vec3::Y),
        ..Default::default()
    });

    // Load an OBJ exported from Truck (e.g., teapot.obj or cube.obj).
    let mesh_handle: Handle<Mesh> = asset_server.load("output/mesh.obj");
    let material = materials.add(StandardMaterial {
        base_color: Color::srgb(0.78, 0.78, 0.78),
        perceptual_roughness: 0.35,
        metallic: 0.05,
        ..Default::default()
    });

    commands.spawn(PbrBundle {
        mesh: mesh_handle,
        material,
        ..Default::default()
    });
}
```

Replace `"output/mesh.obj"` with any OBJ generated in previous chapters (remember it should live under `assets/`). Bevy will hot-reload assets placed in `assets/` when the `file_watcher` feature is enabled.

## Run

```bash
cargo run --bin viewer
```

A lit, shaded version of your mesh should appear immediately.
