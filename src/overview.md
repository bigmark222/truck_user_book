# Overview

Welcome to Truck, a pure-Rust CAD kernel with modern, modular tools for geometric modeling. This book walks through meshes, B-reps, and rendering with Truck.

## The three core ideas

1. **Trendy tools**: Rust for safety/performance; WebGPU for fast, cross-platform graphics and compute.
2. **Traditional arts (modernized)**: Re-implements classic CAD concepts (B-rep, NURBS) with Rust/WebGPU for stability and testability.
3. **Theseus’ Ship (modular design)**: Small, replaceable crates so you can swap or evolve parts independently without breaking the whole system.

## Who this book is for

- **Users**: Build apps with Truck and other libraries.
- **Developers**: Create new geometric tools/elements on Truck.
- **Contributors**: Contribute directly to Truck’s codebase.

This tutorial focuses primarily on Users.

## Sample code

All examples live at [github.com/ricosjp/truck-tutorial-code/tree/v0.1](https://github.com/ricosjp/truck-tutorial-code/tree/v0.1)

- `src/section2_1.rs` corresponds to Section 2.1 “First Triangle”.
- Run with: `cargo run --bin section2_1`

## System requirements

- Rust toolchain
- WebGPU-compatible backend: Vulkan, Metal, or DirectX12
- CMake (only for running tests)

### Windows

- Keep Windows 10 up to date.
- Install Visual Studio C++ Build Tools (update if already installed).
- Install Rust: https://www.rust-lang.org/tools/install (choose MSVC toolchain; `rustup update` if already installed).
- Notes: Tested with MSVC. Vulkan/DirectX12 should work on current Windows 10. WSL lacks Vulkan; Windows 8 likely unsupported.

### macOS

- Keep macOS up to date.
- Install Rust: `curl https://sh.rustup.rs -sSf | sh` (or `rustup update` if installed).
- Notes: Metal works out of the box; no extra GPU setup needed.

### Linux

- Not officially supported yet (limited testing).
- You can try installing Vulkan manually; see https://vulkan.lunarg.com/doc/sdk/1.2.162.1/linux/getting_started_ubuntu.html
- CI builds use the `nvidia/vulkan` Docker image; official containers didn’t work for testing.
