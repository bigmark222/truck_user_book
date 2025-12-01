# Bevy Setup

Set up Bevy and open a blank window.

## Dependencies (`Cargo.toml`)

```toml
[dependencies]
bevy = { version = "0.13", features = ["file_watcher"] }

[dev-dependencies]
cargo-watch = "8" # optional: rebuild/reload when meshes change
```

> Tip: replace `0.13` with the latest version of `bevy` if a newer one is available.

## Minimal app

```rust
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(WindowPlugin {
            primary_window: Some(Window {
                title: "Truck + Bevy".into(),
                ..Default::default()
            }),
            ..Default::default()
        }))
        .add_systems(Startup, setup)
        .run();
}

fn setup(mut commands: Commands) {
    commands.insert_resource(ClearColor(Color::srgb(0.05, 0.05, 0.08)));
    commands.spawn(Camera3dBundle::default());
}
```

## Run it

```bash
cargo run
```

You should see an empty dark window. If it opens, Bevy is ready for rendering Truck meshes.
