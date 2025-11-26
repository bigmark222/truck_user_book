# Overview

Welcome to Truck, a pure-Rust [CAD kernel](https://en.wikipedia.org/wiki/Geometric_modeling_kernel) with modern, modular tools for geometric modeling. This book walks through [meshes](https://en.wikipedia.org/wiki/Mesh_generation), [B-reps](https://en.wikipedia.org/wiki/Boundary_representation), and [rendering](https://en.wikipedia.org/wiki/Rendering_(computer_graphics)) with Truck.

## The three core ideas

1. **Trendy tools**: [Rust](https://en.wikipedia.org/wiki/Rust_(programming_language)) for safety/performance; [WebGPU](https://en.wikipedia.org/wiki/WebGPU) via Rust's [three-d](https://github.com/asny/three-d) library for fast, cross-platform graphics and compute.
2. **Traditional arts (modernized)**: Re-implements classic CAD concepts (B-rep, [NURBS](https://en.wikipedia.org/wiki/Non-uniform_rational_B-spline)) with Rust for stability and testability.
3. **Modular Design ([Theseus’ Ship](https://en.wikipedia.org/wiki/Ship_of_Theseus))**: Small, replaceable crates so you can swap or evolve parts independently without breaking the whole system.

## Who this book is for

- **Users**: Build apps with Truck and other libraries.
- **Developers**: Create new geometric tools/elements on Truck.
- **Contributors**: Contribute directly to Truck’s codebase.

This tutorial focuses primarily on **Users**.

## System requirements

- Rust toolchain (install via [rustup.rs](https://rustup.rs))
- WebGPU-compatible backend:
  - [Vulkan](https://www.vulkan.org) (Windows/Linux)
  - [Metal](https://developer.apple.com/metal/) (macOS)
  - [DirectX12](https://support.microsoft.com/en-us/topic/how-to-install-the-latest-version-of-directx-d1f5ffa5-dae2-246c-91b1-ee1e973ed8c2) (Windows)
- [CMake](https://cmake.org) (only for running tests)

### Windows

- Keep Windows 10 up to date.
- Install [Visual Studio C++ Build Tools](https://visualstudio.microsoft.com/downloads/) (update if already installed).
- Install Rust: [rustup.rs](https://rustup.rs) (choose MSVC toolchain; `rustup update` if already installed).
- Notes: Tested with MSVC. Vulkan/DirectX12 should work on current Windows 10. [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) lacks Vulkan; Windows 8 likely unsupported.

### macOS

- Keep macOS up to date.
- Install Rust: [rustup.rs](https://rustup.rs) (or `rustup update` if installed).
- Notes: Metal works out of the box; no extra GPU setup needed.

### Linux

- Not officially supported yet (limited testing).
- You can try [installing Vulkan manually](https://vulkan.lunarg.com/doc/sdk/1.2.162.1/linux/getting_started_ubuntu.html).
- CI builds use the `nvidia/vulkan` Docker image; official containers didn’t work for testing.
