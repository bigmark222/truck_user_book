Significance of normals as vertex attributes

Up to the previous section, we have displayed polyhedrons, but many of the 3D shapes we usually see are rounded. On the other hand, since meshes are often used to display 3D shapes, rounded shapes are approximated by meshes. If the approximated mesh is displayed as it is, the sphere will appear as if it were a mirror ball. Graphics arbitrarily changes the normals during radiance calculation to achieve a smooth curved surface. Specifically, the data of the normals are registered to the vertices of the mesh, and the points on the triangular surface are calculated by linear interpolation of them to obtain values.

This is a bit difficult to explain, so we will explain it step by step: In 3D graphics, the color of each point on a mesh is calculated and rendered pixel by pixel based on a three-dimensional physical model The simplest optical model used in 3D graphics is the Lambert model. In the Lambert model, the brightness (radiance) of each point on a pure white object surface is expressed as max(-l.dot(n), 0) using the light incident vector l and the surface normal n. In this calculation, there is no rule that the normal of the triangle on the mesh must be used. For example, if you are drawing a mesh that approximates a sphere, such as a mirror ball, it is more appropriate to use the spherical normals at points on the approximated sphere than the triangles. In reality, the spherical information is lost when the surface is meshed, so the spherical normals are registered at the mesh vertices and the normals at the other points on the mesh are linearly interpolated.

If no normals are registered for the mesh, the viewer algorithm completes the normals: the Windows standard viewer and Paraview use the normals of the polygon as is, while the MacOS standard viewer tries to draw the mesh smoothly by averaging the normals of the faces adjacent to the vertices. If you run the section2_4.rs code with the normal completion line commented out as shown below, there should be a difference in edge clarity between the Windows standard viewer and the Mac standard viewer when the output is displayed.

fn write_polyhedron(mut polygon: PolygonMesh, path: &str) {
    // create output obj file
    let mut obj = std::fs::File::create(path).unwrap();
    // add a normal to polyhedron. This will explained in the later sections.
    // polygon.add_naive_normals(true);
    // output polygon to obj file.
    obj::write(&polygon, &mut obj).unwrap();
}
Creation of a mesh with normals

Now that you have learned the significance of registering normals to a mesh, let's actually create a mesh sphere with normals registered. truck has some normal completion algorithms, but if exact values exist, directly assigning those values will produce a more accurate drawing.

While meshing a spherical surface can be done using latitude and longitude, here the mesh is created by inflating a regular hexahedron. This way, the area ratio of the mesh is not too large and the shape can be expressed using only a rectangular mesh. First, let's divert the saving functions for the regular hexahedron created in the previous section and the mesh without normal completion created even earlier.

use std::iter::FromIterator;
use truck_meshalgo::prelude::*;

/// Output the contents of `polygon` to the file specified by `path`.
fn write_polygon(polygon: &PolygonMesh, path: &str) {
    // create output obj file
    let mut obj = std::fs::File::create(path).unwrap();
    // output polygon to obj file.
    obj::write(polygon, &mut obj).unwrap();
}

fn hexahedron() -> PolygonMesh {
    let a = f64::sqrt(3.0) / 3.0;
    let positions = vec![
        Point3::new(-a, -a, -a),
        Point3::new(a, -a, -a),
        Point3::new(a, a, -a),
        Point3::new(-a, a, -a),
        Point3::new(-a, -a, a),
        Point3::new(a, -a, a),
        Point3::new(a, a, a),
        Point3::new(-a, a, a),
    ];
    let attrs = StandardAttributes {
        positions,
        ..Default::default()
    };
    let faces = Faces::from_iter([
        [3, 2, 1, 0],
        [0, 1, 5, 4],
        [1, 2, 6, 5],
        [2, 3, 7, 6],
        [3, 0, 4, 7],
        [4, 5, 6, 7],
    ]);
    PolygonMesh::new(attrs, faces)
}
In the following, we write the main function directly. By dividing each edge of the regular hexahedron into 8 parts, each face is divided into 64 parts, and by projecting the vertices onto the unit sphere, we can create the vertices of a mesh like a mirror ball.

    // create hexahedron
    let hexa = hexahedron();
    // edge parts
    const DIVISION: usize = 8;
    // the positions of vertices
    let positions: Vec<Point3> = hexa
        // for each face of the hexahedron
        .face_iter()
        .flat_map(|face| {
            // each vertex into a vector
            let v: Vec<Vector3> = face
                .iter()
                .map(|vertex| hexa.positions()[vertex.pos].to_vec())
                .collect();
            // create lattice of the square
            (0..=DIVISION)
                .flat_map(move |i| (0..=DIVISION).map(move |j| (i, j)))
                // each i, j runs from 0 to `DIVISION`
                .map(move |(i, j)| {
                    let s = i as f64 / DIVISION as f64;
                    let t = j as f64 / DIVISION as f64;
                    // 線形補間により正方形の格子点を求める
                    v[0] * (1.0 - s) * (1.0 - t)
                        + v[1] * s * (1.0 - t)
                        + v[3] * (1.0 - s) * t
                        + v[2] * s * t
                })
        })
        // noramalize for projecting onto the unit sphere
        .map(|vec| Point3::from_vec(vec.normalize()))
        .collect();
The normal of each point on the unit sphere is obtained by vectorizing the coordinates of that point as is.

    // the sets of all normals
    let normals = positions.iter().copied().map(Point3::to_vec).collect();
Creates a vertex information structure in which vertex coordinates and normals are registered.

    // this time, register the coordinates and normals
    let attrs = StandardAttributes {
        positions,
        normals,
        ..Default::default()
    };
Let us create a surface. This time we also have normals, so we need to set the coordinates and normal indices for the vertices. This time, the coordinates and normals happen to have the same index, but there are cases where different normals correspond to the same coordinate vertex, so the coordinates and normals should be registered separately.


    // the set of faces
    let faces: Faces = (0..6)
        // divide each faces
        .flat_map(|face_idx| {
            // the index of the first vertex on the `face_idx`th face
            let base = face_idx * (DIVISION + 1) * (DIVISION + 1);
            // the closure returns the index of the (i, j) vertex on the square
            let to_index = move |i: usize, j: usize| {
                let idx = base + (DIVISION + 1) * i + j;
                // (index of position, None, Some(index of normal))
                // The middle component is texture. We do not treat texture in this tutorial.
                (idx, None, Some(idx))
            };
            // construct mesh with subdivisions of the square
            (0..DIVISION)
                .flat_map(move |i| (0..DIVISION).map(move |j| (i, j)))
                .map(move |(i, j)| {
                    [
                        to_index(i, j),
                        to_index(i + 1, j),
                        to_index(i + 1, j + 1),
                        to_index(i, j + 1),
                    ]
                })
        })
        .collect();
Now we are ready to go. Let us create a mesh and export it!

    let sphere = PolygonMesh::new(attrs, faces);
    write_polygon(&sphere, "sphere.obj");