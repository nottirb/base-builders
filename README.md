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

## Some notes

### Rust

#### Sandboxing your environment:

Some projects have a `build.rs` file or potentially other issues which can make injecting a `lib.rs` file, or otherwise including it as a dependency a problem. To counteract this problem, you can sandbox the environment as follows:

```Dockerfile
RUN mkdir sandbox \
    && mv Cargo.toml sandbox/Cargo.toml \
    && mv src sandbox/src \
    && mv fuzz sandbox/fuzz

RUN cd ./sandbox/fuzz && cargo +nightly hfuzz build
```

Note that this is a naive implementation and might need to be modified for your use case.

#### Injecting a lib.rs file:

Some projects only include a `main.rs` file. Let's say that you want to access the module `parser` of the project `ex`, you can inject a lib.rs file as follows:

Create a lib.rs file under a folder named inject:
```rs
pub mod parser;
```

```Dockerfile
RUN mv inject/lib.rs src/lib.rs
```

Note that this is a naive implementation and might need to be modified for your use case.

#### Making private functions public:

Some projects have private functions which could be fuzzed, but we want to avoid modifying the source code. All base-builders come with `sed` installed, which we can use to make private functions/modules public.

For instance, let's say that the parser module (`src/parser.rs`) has a function named parse (`fn parse(s: &str)`), which is private. We can make this public as follows:

```Dockerfile
RUN sed -i 's/fn parse/pub fn parse/' ./src/parser.rs \
```

#### A full example (json-parser):

Fuzz harness:
```rs
use honggfuzz::fuzz;
use json_parser::parser::parse;

fn main() {
    loop {
        fuzz!(|data: &[u8]| {
            let json: &str = std::str::from_utf8(data).unwrap(); 
            let _ = parse(json);
        });
    }
}
```

Fuzz Cargo.toml:
```toml
[package]
name = "json-parser-fuzz"
version = "0.0.0"
authors = ["Automatically generated"]
publish = false
edition = "2018"

[package.metadata]
cargo-fuzz = true

[dependencies]
honggfuzz = "0.5.55"

[dependencies.json-parser]
path = ".."

# Prevent this from interfering with workspaces
[workspace]
members = ["."]

[[bin]]
name = "parse"
path = "fuzz_targets/parse.rs"
test = false
doc = false
```

injected lib.rs:
```rs
// Expose all modules
pub mod lexer;
pub mod parser;
```

Dockerfile:
```Dockerfile
# ARGS
ARG project=json-parser


# Setup Fuzz-Rust Builder
FROM ghcr.io/nottirb/fuzz-rust:latest as builder
ADD . /${project}
WORKDIR /${project}

# Sandbox the environment, inject a lib.rs file, expose the parse function, and build the fuzz targets
RUN mkdir sandbox \
    && mv Cargo.toml sandbox/Cargo.toml \
    && mv src sandbox/src \
    && mv inject/lib.rs sandbox/src/lib.rs \
    && mv fuzz sandbox/fuzz \
    && sed -i 's/fn parse/pub fn parse/' ./sandbox/src/parser.rs \
    && cd ./sandbox/fuzz && cargo +nightly hfuzz build


# Setup Fuzz-Env
FROM ghcr.io/nottirb/fuzz-env:latest
COPY --from=builder ${project}/sandbox/fuzz/hfuzz_target/x86_64-unknown-linux-gnu/release/parse /
```
