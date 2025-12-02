# Overview

Welcome to Truck, a pure-Rust [CAD kernel](https://en.wikipedia.org/wiki/Geometric_modeling_kernel) with modern, modular tools for geometric modeling. This book walks through [meshes](https://en.wikipedia.org/wiki/Mesh_generation) and [B-reps](https://en.wikipedia.org/wiki/Boundary_representation) with Truck.

## The three core ideas

1. **Trendy tools**: [Rust](https://en.wikipedia.org/wiki/Rust_(programming_language)) for safety/performance; Truck’s mesh and B-rep crates for modern CAD workflows.
2. **Traditional arts (modernized)**: Re-implements classic CAD concepts (B-rep, [NURBS](https://en.wikipedia.org/wiki/Non-uniform_rational_B-spline)) with Rust for stability and testability.
3. **Modular Design ([Theseus’ Ship](https://en.wikipedia.org/wiki/Ship_of_Theseus))**: Small, replaceable crates so you can swap or evolve parts independently without breaking the whole system.

## Who this book is for

- **Users**: Build apps with Truck and other libraries.
- **Developers**: Create new geometric tools/elements on Truck.
- **Contributors**: Contribute directly to Truck’s codebase.

This tutorial focuses primarily on **Users**.

## System requirements

- Rust toolchain (install via [rustup.rs](https://rustup.rs))
- [CMake](https://cmake.org) (only for running tests)

### Windows

- Keep Windows 10 up to date.
- Install [Visual Studio C++ Build Tools](https://visualstudio.microsoft.com/downloads/) (update if already installed).
- Install Rust: [rustup.rs](https://rustup.rs) (choose MSVC toolchain; `rustup update` if already installed).
- Notes: Tested with MSVC. Windows 8 is likely unsupported.

### macOS

- Keep macOS up to date.
- Install Rust: [rustup.rs](https://rustup.rs) (or `rustup update` if installed).
- Notes: No additional GPU setup required for the chapters covered here.

### Linux

- Not officially supported yet (limited testing).
- CI builds use Linux containers; official support is still evolving.
