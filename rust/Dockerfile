FROM rust:1.47-buster as builder
# Crashing....
# FROM rustlang/rust:nightly-buster as builder

# Because error[E0554]: `#![feature]` may not be used on the stable release channel
# /build/arrow/rust/arrow/src/lib.rs:127:1
# #![feature(specialization)]
RUN rustup update && \
    rustup default nightly-2020-10-25 && \
    rustup component add --toolchain nightly-2020-10-25 rustfmt

RUN apt update && apt upgrade -y && apt install -y git llvm-dev libclang-dev clang

WORKDIR /usr/src

WORKDIR /build/cubestore
COPY Cargo.toml .
COPY Cargo.lock .
COPY cubestore/Cargo.toml cubestore/Cargo.toml
RUN mkdir -p cubestore/src/bin && \
    echo "fn main() {print!(\"Dummy main\");} // dummy file" > cubestore/src/bin/cubestored.rs
RUN RUSTFLAGS="-C target-feature=+avx2" cargo build --release

COPY cubestore .
RUN RUSTFLAGS="-C target-feature=+avx2" cargo build --release

FROM debian:buster-slim

WORKDIR /cube

RUN set -ex; \
	apt-get update; \
	apt-get install -y libssl1.1

COPY --from=builder /build/cubestore/target/release/cubestored .

EXPOSE 3306

CMD ["./cubestored"]
