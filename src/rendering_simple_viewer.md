# Reusable Viewer

Build a reusable Bevy viewer you can run against any OBJ coming out of the earlier chapters. It adds orbit controls, zoom, drag-and-drop, and asset hot reload.

## Viewer structure

- Accepts a mesh path via CLI (`cargo run -- output/mesh.obj`).
- Loads the OBJ as a PBR mesh (default OBJ loader).
- Orbit + scroll for navigation (reuse the orbit system from the previous section).
- Drag-and-drop to swap meshes without restarting.
- Asset hot reload enabled so edits under `assets/` refresh in-place.

### Split into `lib.rs`, `main.rs`, `app.rs`, and `sync.rs`

- `lib.rs` re-exports the Bevy app builder and a `run_with_mesh` helper for downstream crates (like `truck_ui`).
- `main.rs` wires up CLI args and launches the viewer app.
- `app.rs` owns Bevy plugins, cameras, lights, and viewer systems.
- `sync.rs` is a thin bridge for editor <-> viewer data (selection, commands); it stays in the viewer crate so Bevy/Rapier internals remain isolated from UI code.

`truck_viewer/src/lib.rs`

```rust
pub mod app;
pub mod sync;

pub use app::build_app;

pub fn run_with_mesh(mesh_path: impl Into<String>) {
    build_app(mesh_path.into()).run();
}
```

`truck_viewer/src/main.rs`

```rust
fn main() {
    let mesh_path = std::env::args()
        .nth(1)
        .unwrap_or_else(|| "output/mesh.obj".into());

    truck_viewer::run_with_mesh(mesh_path);
}
```

`truck_viewer/src/app.rs`

```rust
use bevy::asset::{AssetPlugin, ChangeWatcher};
use bevy::input::drag_and_drop::FileDragAndDrop;
use bevy::input::mouse::{MouseMotion, MouseWheel};
use bevy::prelude::*;
use std::time::Duration;

use crate::sync::{publish_selection, SelectionState};

#[derive(Resource, Clone)]
struct MeshPath(String);

#[derive(Resource)]
struct OrbitCamera {
    yaw: f32,
    pitch: f32,
    radius: f32,
    target: Vec3,
}

#[derive(Component)]
struct ModelRoot;

pub fn build_app(mesh_path: String) -> App {
    let mut app = App::new();
    app.insert_resource(MeshPath(mesh_path))
        .insert_resource(OrbitCamera {
            yaw: 0.0,
            pitch: 0.3,
            radius: 6.0,
            target: Vec3::ZERO,
        })
        .insert_resource(SelectionState::default())
        .add_plugins(DefaultPlugins.set(AssetPlugin {
            watch_for_changes: ChangeWatcher::with_delay(Duration::from_millis(200)),
            ..Default::default()
        }))
        .add_systems(Startup, setup)
        .add_systems(Update, (orbit_camera, reload_on_drop, publish_selection));
    app
}

fn setup(
    mut commands: Commands,
    mesh_path: Res<MeshPath>,
    asset_server: Res<AssetServer>,
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    commands.insert_resource(ClearColor(Color::srgb(0.05, 0.05, 0.08)));
    commands.insert_resource(AmbientLight {
        color: Color::WHITE,
        brightness: 0.25,
    });

    commands.spawn(Camera3dBundle {
        transform: Transform::from_xyz(3.0, 3.0, 6.0).looking_at(Vec3::ZERO, Vec3::Y),
        ..Default::default()
    });
    commands.spawn(DirectionalLightBundle {
        directional_light: DirectionalLight {
            illuminance: 10_000.0,
            shadows_enabled: true,
            ..Default::default()
        },
        transform: Transform::from_xyz(1.0, -1.0, -1.5).looking_at(Vec3::ZERO, Vec3::Y),
        ..Default::default()
    });

    spawn_model(&mut commands, &asset_server, &mut materials, &mesh_path);
}

fn spawn_model(
    commands: &mut Commands,
    asset_server: &Res<AssetServer>,
    materials: &mut ResMut<Assets<StandardMaterial>>,
    mesh_path: &MeshPath,
) {
    let mesh: Handle<Mesh> = asset_server.load(mesh_path.0.as_str());
    let material = materials.add(StandardMaterial {
        base_color: Color::srgb(0.75, 0.75, 0.75),
        perceptual_roughness: 0.32,
        metallic: 0.05,
        ..Default::default()
    });

    commands.spawn((
        PbrBundle {
            mesh,
            material,
            ..Default::default()
        },
        ModelRoot,
    ));
}

fn reload_on_drop(
    mut commands: Commands,
    mut events: EventReader<FileDragAndDrop>,
    mut mesh_path: ResMut<MeshPath>,
    asset_server: Res<AssetServer>,
    mut materials: ResMut<Assets<StandardMaterial>>,
    existing: Query<Entity, With<ModelRoot>>,
) {
    for event in events.read() {
        if let FileDragAndDrop::DroppedFile { path_buf, .. } = event {
            mesh_path.0 = path_buf.to_string_lossy().to_string();
            for entity in existing.iter() {
                commands.entity(entity).despawn_recursive();
            }
            spawn_model(&mut commands, &asset_server, &mut materials, &mesh_path);
        }
    }
}

fn orbit_camera(
    mouse_buttons: Res<Input<MouseButton>>,
    mut mouse_motion_events: EventReader<MouseMotion>,
    mut mouse_wheel_events: EventReader<MouseWheel>,
    mut orbit: ResMut<OrbitCamera>,
    mut query: Query<&mut Transform, With<Camera>>,
) {
    if mouse_buttons.pressed(MouseButton::Left) {
        let mut delta = Vec2::ZERO;
        for ev in mouse_motion_events.read() {
            delta += ev.delta;
        }
        orbit.yaw -= 0.0025 * delta.x;
        orbit.pitch = (orbit.pitch - 0.002 * delta.y).clamp(-1.5, 1.5);
    }

    for ev in mouse_wheel_events.read() {
        orbit.radius = (orbit.radius - ev.y * 0.5).clamp(1.5, 50.0);
    }

    let rotation =
        Quat::from_axis_angle(Vec3::Y, orbit.yaw) * Quat::from_axis_angle(Vec3::X, orbit.pitch);
    let translation = rotation * Vec3::new(0.0, 0.0, orbit.radius);

    for mut transform in &mut query {
        transform.translation = orbit.target + translation;
        transform.look_at(orbit.target, Vec3::Y);
    }
}
```

`truck_viewer/src/sync.rs`

```rust
use bevy::prelude::*;

/// Minimal shared state that other crates (like `truck_ui`) can read without
/// depending on Bevy internals.
#[derive(Resource, Default, Clone)]
pub struct SelectionState {
    pub focused_entity: Option<Entity>,
}

/// Publish selection or other viewer state to external listeners.
pub fn publish_selection(selection: Res<SelectionState>) {
    // In a full app, send this over a channel/observer API to truck_ui.
    // Here it is a placeholder to show where sync code lives.
    let _snapshot = selection.clone();
    let _ = _snapshot;
}
```

## Run with auto-reload

Place your meshes under `assets/` (for example `assets/output/mesh.obj` on disk; the Bevy path is `output/mesh.obj`). Then:

```bash
cargo run -- output/mesh.obj
```

- Drag another OBJ onto the window to swap models instantly.
- Leave `file_watcher` enabled for automatic asset reloads; pair with `cargo watch -x run` if you also want code rebuilds on change.
