# earthly-nix

> An Earthfile for installing and using Nix development shells

[Nix] is a popular package manager which boasts tens of thousands of [packages].
When combined with [development shells], it provides reproducible development
environments where dependencies can be easily identified and pinned.

[Earthly] provides a friendly UX on top of [Docker] and [Buildkit]. It allows
defining targets (similar to [make]) which can be executed the same locally as
they are in CI. Additionally, with Earthly satellites, caching is given
first-class treatment resulting in a significant decrease in build times.

Why combine the two? Development shells are unmatched in terms of developer
productivity, especially across large teams. However, utilizing them requires
installing Nix on your local system, which can be very intrusive. By
encapsulating the Nix development shell into Earthly, it's possible to gain the
benefits of clearly identifying/locking development dependencies with _very
little_ modification of local systems (just Docker running and the Earthly CLI
installed).

The additional layer of isolation provided by Earthly also makes it much easier
to reason about what is going on in CI. The CI systems simply invoke Earthly
targets which will run in the exact same manner as they would locally. With
Earthly satellites, the whole process can be lifted to hosted runners with
first class support for caching.

## Usage

This repository provides two things:

1. A base image (debian slim) which installs the latest version of Nix
2. A [UDC] which loads the default development shell from a `flake.nix`

Additionally, it presumes the following structure in your local repository:

- nix/**
- flake.nix
- flake.lock

Where `nix/**` is a directory where dependent nix code is located. The UDT will
automatically copy these files into the container and load the default
development shell.

A typical Earthfile might look like:

```Earthfile
VERSION 0.7

nix:
    FROM github.com/jmgilman/earthly-nix+nix
    DO github.com/jmgilman/earthly-nix+SETUP

test:
    FROM +nix
    RUN with_nix treefmt
```

In the first target, we inherit from the `nix` target which contains all of the
Nix tooling. After that, we call the `SETUP` UDC which loads the default
development shell from your local repository. Finally, it writes a `with_nix`
script which can be used for calling nix dependencies. In the above example,
we call the `treefmt` CLI which was supplied by our development shell.

## Why not build with Nix?

Building with Nix is still very much possible. The benefit that Earthly provides
is not requiring a Nix installation on your local system. For solo developers
using Nix, the extra layer might not make sense. Where it excels is in an
open-source project where external contributors might not be interested in
setting up Nix.

[buildkit]: https://docs.docker.com/build/buildkit/
[development shells]: https://github.com/numtide/devshell
[docker]: https://www.docker.com/
[earthly]: https://earthly.dev/
[make]: https://www.gnu.org/software/make/manual/make.html
[nix]: https://nixos.org/
[packages]: https://search.nixos.org/packages
[udc]: https://docs.earthly.dev/docs/guides/udc