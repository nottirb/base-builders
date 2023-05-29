# Base Fuzz Builders

Docker images to use in fuzzing repositories. Images are built and published daily.

## Default Fuzzing Environment

Features all dependencies for running a fuzzer. Uses `ubuntu:20.04`.

```Dockerfile
FROM ghcr.io/nottirb/fuzz-env
```

## Rust

Features: `cargo-fuzz`, `afl`, `honggfuzz`

```Dockerfile
FROM ghcr.io/nottirb/fuzz-rust
```