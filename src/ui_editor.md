# UI and Editor (`truck_ui`)

The editor lives in its own crate (`truck_ui`) and sits on top of the `truck_viewer` runtime (Bevy + Rapier). Keep Bevy and Rapier together so they can share the same ECS world; use clear APIs/events for UI to drive the runtime without reaching into internals.

## What `truck_ui` owns

- Tools, menus, and the feature tree (sketches, features, timeline).
- Selection, gizmos, and manipulation modes that call into `truck_viewer`.
- Command stack/undo, with commands expressed in terms of viewer actions (spawn, move, delete, apply material).
- File/session management and autosave; UI state that is independent of the scene ECS.

## How it talks to `truck_viewer`

- Expose a slim interface from `truck_viewer`: start/stop the app, load meshes/breps, spawn bodies, apply transforms/materials, subscribe to selection hits.
- Send events into the ECS instead of mutating components directly; keep the boundary testable.
- Listen for viewer signals (asset reloads, physics contacts, selection rays) and map them to UI updates.

## Target crate layout (after Chapter 6)

```
truck_ui/                  # editor logic (no Bevy/Rapier here)
├── Cargo.toml
└── src/
    ├── lib.rs
    ├── commands.rs        # enums for operations / tools
    └── state.rs           # EditorState, selection, history, etc.
```

## Integration checkpoints

- **After Chapter 4**: UI can launch the viewer, load a Truck OBJ, and orbit/pan/zoom the camera.
- **After Chapter 5**: UI toggles physics, spawns bodies with colliders, and shows debug render overlays.
- **Capstone**: Hook up a couple of editor tools (e.g., move + rotate) that emit viewer events and confirm selection feedback.
