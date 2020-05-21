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

# This should be executed from the project root.

set -e

mkdir -p target

project=${PWD##*/} # Use the directory's name as the name of the project.
container="$project-freebsd-build"

# Use the container itself as the cache layer for the registry.
# This implies we won't create a new container for every build,
# but just start the same container.
if ! docker ps -a --format '{{.Names}}' | grep -Eq "^${container}\$"; then
	docker create \
	       --name "$container" \
	       -i -a STDIN -a STDOUT -a STDERR \
	       -v "$(pwd)":/rust/project:ro \
	       freebsd-cross-rust
fi

echo "cargo build --release --target x86_64-unknown-freebsd --target-dir /rust/target" \
	| docker start -i "$container"

docker cp "$container":/rust/target/x86_64-unknown-freebsd/ target/x86_64-unknown-freebsd/
```

The script sets up a container for building your project. The build artefacts are cached
within the container, and the build is executed by just piping the build command into the
container. As with the build command, other commands can be piped into the container:
- To install possible dependencies, like `libfl-dev` and `libsqlite3-dev`.
- To install other toolchains, like `nightly`.

Please note that when installing a new toolchain, the `x86_64-unknown-freebsd` target must
be added for such toolchain.
