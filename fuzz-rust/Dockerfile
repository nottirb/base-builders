FROM rustlang/rust:nightly

RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y build-essential clang-11 llvm-11-tools cmake clang curl binutils-dev libbfd-dev libunwind8-dev libblocksruntime-dev pkg-config libudev-dev libssl-dev libgc-dev libtool-bin autoconf && \
    ln -s /usr/bin/llvm-config-11 /usr/bin/llvm-config

RUN rustup component add rust-src
RUN cargo install -f cargo-fuzz
RUN cargo install -f afl
RUN cargo install -f honggfuzz
