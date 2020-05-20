# freebsd-cross-build

This creates a container that can be used to cross-compile FreeBSD binaries
natively on Linux.

To build:

    docker build -t freebsd-cross .

To run:

    docker run --rm -it freebsd-cross bash

There is `/freebsd/bin` on the path. It has all of the build
tools (e.g. x86_64-pc-freebsd12-gcc)

It is likely you would add a `-v` switch to the run (to put your
code on a mount).

## Rust Image

There is a second Dockerfile that can be used to cross-compile FreeBSD
Rust binaries.

To build:

    docker build -f Dockerfile.rust -t freebsd-cross-rust .

I wrote [a blog post about creating this on my blog](https://www.wezm.net/technical/2019/03/cross-compile-freebsd-rust-binary-with-docker/).

## Example of its use on a Rust project:
In your project add a `.cargo/config` file for the x86_64-unknown-freebsd target. This tells
cargo what tool to use as the linker.

```toml
[target.x86_64-unknown-freebsd]
linker = "x86_64-pc-freebsd12-gcc"
```

Then, use the following script:

```shell
#!/bin/sh

set -e

mkdir -p target/x86_64-unknown-freebsd

# NOTE: Assumes the following volumes have been created:
# - lobsters-freebsd-target
# - lobsters-freebsd-cargo-registry
# And that there is a .cargo/config present that sets the linker appropriately
# for the x86_64-unknown-freebsd target.

# Build
sudo docker run --rm -it \
  -v "$(pwd)":/home/rust/code:ro \
  -v lobsters-freebsd-target:/home/rust/code/target \
  -v lobsters-freebsd-cargo-registry:/home/rust/.cargo/registry \
  freebsd-cross-rust build --release --target x86_64-unknown-freebsd

# Copy binary out of volume into target/x86_64-unknown-freebsd
sudo docker run --rm -it \
  -v "$(pwd)"/target/x86_64-unknown-freebsd:/home/rust/output \
  -v lobsters-freebsd-target:/home/rust/code/target \
  --entrypoint cp \
  freebsd-cross-rust \
  /home/rust/code/target/x86_64-unknown-freebsd/release/lobsters /home/rust/output
```
