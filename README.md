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

## Full Fuzzing Build/Environment Example (Honggfuzz)

```Dockerfile
# ARGS
ARG project=example


# Setup Fuzz-Rust Builder
FROM ghcr.io/nottirb/fuzz-rust:latest as builder
ADD . /${project}
WORKDIR /${project}
RUN cd ./fuzz && cargo +nightly hfuzz build


# Setup Fuzz-Env
FROM ghcr.io/nottirb/fuzz-env:latest
COPY --from=builder ${project}/fuzz/hfuzz_target/x86_64-unknown-linux-gnu/release/fuzz_target_1 /
COPY --from=builder ${project}/fuzz/hfuzz_target/x86_64-unknown-linux-gnu/release/fuzz_target_2 /
```