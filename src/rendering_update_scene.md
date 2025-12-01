# Orbit + Hot Reload

Add basic orbit controls and make the viewer reload whenever the OBJ file changes.

## Accept a mesh path

Let the binary remain tiny and read a file path (defaulting to `assets/output/mesh.obj`) before calling into the library:

```rust
let mesh_path = std::env::args()
    .nth(1)
    .unwrap_or_else(|| "output/mesh.obj".into());
```

Store it in a resource so startup systems can load it. Paths are relative to your `assets/` directory (so `output/mesh.obj` means `assets/output/mesh.obj` on disk).

## Orbit + zoom

A simple orbit controller using Bevy input events. Keep it in `truck_viewer/src/lib.rs` (or split into `app.rs` and re-export from `lib.rs`), and have `src/main.rs` just pass the mesh path to a helper like `run_with_mesh`.

```rust
use bevy::prelude::*;
use bevy::input::mouse::{MouseMotion, MouseWheel};

#[derive(Resource)]
pub struct OrbitCamera {
    yaw: f32,
    pitch: f32,
    radius: f32,
    target: Vec3,
}

pub fn build_app(mesh_path: String) -> App {
    let mut app = App::new();
    app.add_plugins(DefaultPlugins.set(WindowPlugin {
        primary_window: Some(Window {
            title: "Truck mesh with Bevy".into(),
            ..Default::default()
        }),
        ..Default::default()
    }))
    .insert_resource(mesh_path)
    .insert_resource(OrbitCamera {
        yaw: 0.0,
        pitch: 0.3,
        radius: 6.0,
        target: Vec3::ZERO,
    })
    .add_systems(Startup, setup_camera)
    .add_systems(Update, orbit_camera);
    app
}

pub fn run_with_mesh(mesh_path: impl Into<String>) {
    build_app(mesh_path.into()).run();
}

fn setup_camera(mut commands: Commands) {
    commands.spawn(Camera3dBundle {
        transform: Transform::from_xyz(0.0, 0.0, 6.0).looking_at(Vec3::ZERO, Vec3::Y),
        ..Default::default()
    });
}

fn orbit_camera(
    mouse_buttons: Res<Input<MouseButton>>,
    mut mouse_motion_events: EventReader<MouseMotion>,
    mut mouse_wheel_events: EventReader<MouseWheel>,
    mut orbit: ResMut<OrbitCamera>,
    mut query: Query<&mut Transform, With<Camera>>,
) {
    // Mouse drag to orbit
    if mouse_buttons.pressed(MouseButton::Left) {
        let mut delta = Vec2::ZERO;
        for ev in mouse_motion_events.read() {
            delta += ev.delta;
        }
        orbit.yaw += delta.x * 0.01;
        orbit.pitch = (orbit.pitch + delta.y * 0.01).clamp(-1.2, 1.2);
    } else {
        mouse_motion_events.clear();
    }

    // Scroll to zoom
    for ev in mouse_wheel_events.read() {
        orbit.radius = (orbit.radius - ev.y * 0.3).clamp(2.0, 20.0);
    }

    // Update camera from spherical coords
    if let Ok(mut transform) = query.get_single_mut() {
        let dir = Vec3::new(
            orbit.yaw.sin() * orbit.pitch.cos(),
            orbit.pitch.sin(),
            orbit.yaw.cos() * orbit.pitch.cos(),
        );
        let eye = orbit.target + dir * orbit.radius;
        transform.translation = eye;
        transform.look_at(orbit.target, Vec3::Y);
    }
}
```

`truck_viewer/src/main.rs` can stay minimal:

```rust
fn main() {
    let mesh_path = std::env::args()
        .nth(1)
        .unwrap_or_else(|| "output/mesh.obj".into());

    truck_viewer::run_with_mesh(mesh_path);
}
```

## Auto-reload with `cargo watch`

Install once:

```bash
cargo install cargo-watch
```

Run the viewer and rerun on file changes (so your mesh reloads cleanly):

```bash
cargo watch -w assets/output/mesh.obj -x 'run -- output/mesh.obj'
```

If you keep Bevy’s `file_watcher` feature enabled, asset edits in `assets/` will hot-reload without restarting; the `cargo watch` loop is still handy for rebuilding code changes alongside mesh updates.
