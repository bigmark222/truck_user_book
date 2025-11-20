# Updated Scene

Animate the camera and multiple lights around a teapot to create a dynamic scene.

Reference: [github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter4/src/section4_3.rs](https://github.com/ricosjp/truck-tutorial-code/blob/v0.6/chapter4/src/section4_3.rs)

## Imports

```rust
mod app;
use app::*;
use truck_platform::*;
use truck_rendimpl::*;
use winit::window::Window;
use std::sync::Arc;
use std::f64::consts::PI;
```

## Camera (perspective)

```rust
let camera = Camera::perspective_camera(
    Matrix4::identity(), // view matrix updated each frame
    Rad(PI / 4.5),       // slightly narrow FOV
    0.1,                 // near
    10.0,                // far
);
```

Camera orientation will be animated every frame.

## RGB lights

Place three colored point lights in an equilateral triangle around the teapot.

```rust
let radius = 5.0 * f64::sqrt(2.0);
let omega = [0.5, f64::sqrt(3.0) * 0.5];

let positions = [
    Point3::new(radius * omega[0], 6.0,  radius * omega[1]),
    Point3::new(-radius,           6.0,  0.0),
    Point3::new(radius * omega[0], 6.0, -radius * omega[1]),
];

let lights = vec![
    Light { position: positions[0], color: Vector3::new(1.0, 0.0, 0.0), light_type: LightType::Point },
    Light { position: positions[1], color: Vector3::new(0.0, 1.0, 0.0), light_type: LightType::Point },
    Light { position: positions[2], color: Vector3::new(0.0, 0.0, 1.0), light_type: LightType::Point },
];
```

## Scene descriptor

```rust
let scene_desc = WindowSceneDescriptor {
    studio: StudioConfig {
        camera,
        lights,
        ..Default::default()
    },
    ..Default::default()
};

let mut scene = WindowScene::from_window(window, &scene_desc).await;
```

Load `teapot.obj` and add it just like in the previous section.

## Animation (in `render`)

```rust
let time = self.scene.elapsed().as_secs_f64();

// Access camera + lights
let (camera, lights) = {
    let studio = self.scene.studio_config_mut();
    (&mut studio.camera, &mut studio.lights)
};

// Camera orbit at 1 rad/s
let rot = Matrix4::from_axis_angle(Vector3::unit_y(), Rad(time));
camera.matrix = rot
    * Matrix4::look_at(
        Point3::new(5.0, 6.0, 5.0),
        Point3::new(0.0, 1.5, 0.0),
        Vector3::unit_y(),
    )
    .invert()
    .unwrap();

// Vertical light oscillation
lights[0].position[1] = 6.0 * (time * 0.8).cos();
lights[1].position[1] = -6.0 * (time * 0.8).cos();
lights[2].position[1] = 6.0 * (time * 0.8).cos();
```

Render after updating:

```rust
self.scene.render_scene(&frame.output.view);
```

## Result

- Rotating camera around the teapot
- Three independently moving colored lights
- Dynamic, colorful shading driven by per-frame updates
