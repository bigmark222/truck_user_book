# Truck User Book

MDNbook content for Truck tutorials covering meshes, modeling (B-rep), and rendering. Use `mdbook` to build or serve locally.

## Prerequisites

- Rust toolchain
- `mdbook` (`cargo install mdbook` if needed)

## Usage

```bash
# preview with live reload
mdbook serve -o

# build static site
mdbook build
```

## Structure

- `src/`: chapter content (mesh, modeling, rendering)
- `src/SUMMARY.md`: chapter ordering
- `book/`: generated output (ignored by git)
