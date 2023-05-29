# Base Fuzz Builders

Docker images to use in fuzzing repositories. Images are built and published daily.

## Rust

Features: `cargo-fuzz`, `afl`, `honggfuzz`

```Dockerfile
FROM ghcr.io/nottirb/fuzz-rust
```