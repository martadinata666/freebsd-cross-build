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

See the following script for an example of its use on a Rust project:

<https://git.sr.ht/~wezm/lobsters/tree/master/scripts/build-freebsd>

I wrote [a blog post about creating this on my blog](https://www.wezm.net/technical/2019/03/cross-compile-freebsd-rust-binary-with-docker/).
