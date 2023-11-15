**Wax** is [a portable globbing library](https://github.com/olson-sean-k/wax).
This repository contains the source code for the [documentation
website](https://glob.guide).

The website is built via `make` using
[MkDocs](https://www.mkdocs.org/#installation), the
[Rust](https://www.rust-lang.org/tools/install) nightly toolchain, and
[peru](https://github.com/buildinspace/peru#installation).

The `Makefile` has two primary rules: `build` and `publish`, which do exactly
what their names suggest. Build artifacts are written to the `./out` directory.
When publishing, these artifacts are copied into a temporary directory via
`mktemp`.

The `gh-pages` branch typically has a single commit and is used exclusively for
build artifacts hosted by GitHub Pages (there is no history). Publishing with
`make publish` forces a push to the push specification of the `origin/gh-pages`
branch.

Peru is used to fetch the Wax source code. This code is used to generate API
documentation via `rustdoc`.

It is possible to test changes via `mkdocs serve`, but this will not serve the
`rustdoc` API documentation.
