# Physics With Rapier (In the Viewer)

Pair Bevy with Rapier inside the same `truck_viewer` crate to simulate and test the parts you build in Truck. This chapter covers what the combo can do for engineering and assembly workflows and shows how to bolt physics onto your existing viewer.

## Why Bevy + Rapier?

- **Fast iteration**: Bevy hot-reloads assets (with `file_watcher`) and has an ECS that keeps systems small and composable.
- **Robust physics**: Rapier adds rigid bodies, joints, collisions, CCD, sleeping, mass properties, restitution, friction, and sensors.
- **Engineering-friendly**: Test clearances, gravity fits, tolerances, drop tests, and motion envelopes before you cut metal.
- **Assembly previews**: Wire joints between Truck parts to validate hinges, sliders, constraints, and interference.
- **Debugging tools**: Rapier debug render + Bevy gizmos for visualizing colliders, normals, centers of mass, and contact points.

## Minimal physics viewer (build on Chapter 4)

Add Rapier next to Bevy in `truck_viewer/Cargo.toml`:

```toml
[dependencies]
bevy = { version = "0.17", features = ["file_watcher"] }
bevy_rapier3d = { version = "0.26", features = ["debug-render-3d"] }
```

Then extend the Chapter 4 viewer with physics:

```rust
use bevy::prelude::*;
use bevy_rapier3d::prelude::*;

fn main() {
    App::new()
        .add_plugins((
            DefaultPlugins,
            RapierPhysicsPlugin::<NoUserData>::default(),
            RapierDebugRenderPlugin::default(), // remove if you want a clean view
        ))
        .insert_resource(ClearColor(Color::srgb(0.06, 0.06, 0.08)))
        .add_systems(Startup, (setup_scene, spawn_part))
        .run();
}

fn setup_scene(mut commands: Commands) {
    commands.insert_resource(AmbientLight {
        color: Color::WHITE,
        brightness: 0.2,
    });
    commands.spawn(Camera3dBundle {
        transform: Transform::from_xyz(4.0, 4.0, 8.0).looking_at(Vec3::ZERO, Vec3::Y),
        ..Default::default()
    });
    commands.spawn(DirectionalLightBundle::default());

    // Ground plane so things have something to hit.
    commands.spawn((
        PbrBundle {
            transform: Transform::from_xyz(0.0, -1.0, 0.0)
                .with_scale(Vec3::new(20.0, 0.1, 20.0)),
            ..Default::default()
        },
        Collider::cuboid(10.0, 0.05, 10.0),
    ));
}

fn spawn_part(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    // Swap this path for any Truck OBJ under assets/.
    let mesh: Handle<Mesh> = asset_server.load("output/mesh.obj");
    let material = materials.add(StandardMaterial {
        base_color: Color::srgb(0.78, 0.78, 0.78),
        perceptual_roughness: 0.35,
        metallic: 0.05,
        ..Default::default()
    });

    commands.spawn((
        PbrBundle {
            mesh,
            material,
            transform: Transform::from_xyz(0.0, 3.0, 0.0),
            ..Default::default()
        },
        RigidBody::Dynamic,
        Collider::ball(0.5), // choose a collider matching your part; cuboid/trimesh/convex hull also work
        Restitution {
            coefficient: 0.7,
            combine_rule: CoefficientCombineRule::Max,
        },
        Friction {
            coefficient: 0.5,
            combine_rule: CoefficientCombineRule::Average,
        },
    ));
}
```

### Next steps for assembly simulations

- **Better colliders**: Use convex hulls or trimesh colliders for fidelity; simplify meshes for speed.
- **Joints/constraints**: Add revolute, prismatic, fixed, and ball joints to connect parts and validate degrees of freedom.
- **Sensors**: Use sensor colliders to detect contacts without forces (clearance checks, triggers).
- **CCD and tolerances**: Enable CCD on fast-moving parts to avoid tunneling; tweak friction and restitution to match materials.
- **Profiling**: Turn off `RapierDebugRenderPlugin` when measuring performance; scale physics timestep if needed.

Run it with:

```bash
cargo run --bin viewer
```

Keep your meshes in `assets/` so Bevy’s asset server (and hot reload) works—the `file_watcher` feature picks up asset changes automatically. Use `cargo watch -x run` if you also want code edits to rebuild/restart without manual steps.
