# Simple Viewer

Build a minimal 3D viewer that can display a mesh, zoom, rotate, and load OBJ files via drag-and-drop.

Reference: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter4/src/section4_4.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter4/src/section4_4.rs)

## Imports

```rust
mod app;
use app::*;

use std::sync::Arc;
use std::f64::consts::PI;

use truck_platform::*;
use truck_rendimpl::*;

use winit::{
    dpi::*,
    event::*,
    window::Window,
};
```

## App state

Track the scene, drag state, and last cursor position:

```rust
struct MyApp {
    scene: WindowScene,
    rotate_flag: bool,
    prev_cursor: Vector2,
}
```

## Init: default cube viewer

Set up camera + light, then load a cube (`hexahedron.obj`).

```rust
#[async_trait(?Send)]
impl App for MyApp {
    async fn init(window: Arc<Window>) -> Self {
        let mut scene = WindowScene::from_window(
            window,
            &WindowSceneDescriptor {
                studio: StudioConfig {
                    camera: Camera::perspective_camera(
                        Matrix4::look_at_rh(
                            Point3::new(1.5, 1.5, 1.5),
                            Point3::origin(),
                            Vector3::unit_y(),
                        )
                        .invert()
                        .unwrap(),
                        Rad(PI / 4.0),
                        0.1,
                        40.0,
                    ),
                    lights: vec![Light {
                        position: Point3::new(1.5, 1.5, 1.5),
                        color: Vector3::new(1.0, 1.0, 1.0),
                        light_type: LightType::Point,
                    }],
                    ..Default::default()
                },
                ..Default::default()
            },
        )
        .await;

        let mesh = polymesh::obj::read(include_bytes!("hexahedron.obj").as_slice()).unwrap();

        let instance = scene.instance_creator().create_instance(
            &mesh,
            &PolygonState {
                material: Material {
                    albedo: Vector4::new(0.75, 0.75, 0.75, 1.0),
                    reflectance: 0.2,
                    roughness: 0.2,
                    ambient_ratio: 0.02,
                    ..Default::default()
                },
                ..Default::default()
            },
        );

        scene.add_object(&instance);

        MyApp {
            scene,
            rotate_flag: false,
            prev_cursor: Vector2::zero(),
        }
    }

    fn render(&mut self) {
        self.scene.render_frame()
    }
}
```

## Zoom with mouse wheel

Move the camera along its view direction; keep the light at the camera:

```rust
fn mouse_wheel(&mut self, delta: MouseScrollDelta, _: TouchPhase) -> ControlFlow {
    if let MouseScrollDelta::LineDelta(_, y) = delta {
        let sc_desc = self.scene.descriptor_mut();
        let (camera, light) = (&mut sc_desc.camera, &mut sc_desc.lights[0]);

        let trans = Matrix4::from_translation(camera.eye_direction() * 0.2 * y as f64);
        camera.matrix = trans * camera.matrix;
        light.position = camera.position();
    }
    Self::default_control_flow()
}
```

## Rotate with mouse drag

Toggle on left-button press; rotate on cursor move.

```rust
fn mouse_input(&mut self, state: ElementState, button: MouseButton) -> ControlFlow {
    if let MouseButton::Left = button {
        self.rotate_flag = state == ElementState::Pressed;
    }
    Self::default_control_flow()
}

fn cursor_moved(&mut self, pos: PhysicalPosition<f64>) -> ControlFlow {
    let position = Vector2::new(pos.x, pos.y);

    if self.rotate_flag {
        let desc = self.scene.descriptor_mut();
        let (camera, light) = (&mut desc.camera, &mut desc.lights[0]);

        let dir2d = position - self.prev_cursor;
        if !dir2d.so_small() {
            let axis = (dir2d[1] * camera.matrix[0].truncate()
                + dir2d[0] * camera.matrix[1].truncate())
                .normalize();
            let angle = dir2d.magnitude() * 0.01;
            let mat = Matrix4::from_axis_angle(axis, Rad(-angle));

            camera.matrix = mat * camera.matrix;
            light.position = camera.position();
        }
    }

    self.prev_cursor = position;
    Self::default_control_flow()
}
```

## Drag & drop OBJ files

Load, normalize to unit size at origin, and display.

```rust
fn dropped_file(&mut self, path: std::path::PathBuf) -> ControlFlow {
    // load file
    let obj = match std::fs::read(path) {
        Ok(x) => x,
        Err(e) => {
            eprintln!("{e}");
            return Self::default_control_flow();
        }
    };

    // parse mesh
    let mut mesh = match polymesh::obj::read(obj.as_slice()) {
        Ok(x) => x,
        Err(e) => {
            eprintln!("{e}");
            return Self::default_control_flow();
        }
    };

    // normalize to fit view
    let bbx = mesh.bounding_box();
    let center = bbx.center().to_vec();
    let diameter = bbx.diameter();
    mesh.positions_mut().iter_mut().for_each(|p| {
        *p = (*p - center) / (diameter / 2.0);
    });

    // create instance and display
    let instance = self.scene.instance_creator().create_instance(
        &mesh,
        &PolygonState {
            material: Material {
                albedo: Vector4::new(0.75, 0.75, 0.75, 1.0),
                reflectance: 0.2,
                roughness: 0.2,
                ambient_ratio: 0.02,
                ..Default::default()
            },
            ..Default::default()
        },
    );

    self.scene.clear_objects();
    self.scene.add_object(&instance);

    Self::default_control_flow()
}
```

## Result

- Load OBJ files (drag-and-drop)
- Interactive rotation (left-drag)
- Mouse wheel zoom
- Automatic centering and scaling
- Lit/shaded meshes for quick previews and debugging
