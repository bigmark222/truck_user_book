# [Truck User Book](https://bigmark222.github.io/truck_user_book/)


MDNbook content for Truck tutorials covering meshes and modeling (B-rep). Use `mdbook` to build or serve locally.

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

- `src/`: chapter content (mesh, modeling)
- `src/SUMMARY.md`: chapter ordering
- `book/`: generated output (ignored by git)

**For the full code, refer to the [truck_user_book_code](https://github.com/bigmark222/truck_user_book_code) repo.**
*which is largely based off of work from [RICOS Co. Ltd.](https://github.com/ricosjp).*
