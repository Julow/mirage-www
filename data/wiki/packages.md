---
updated: 2017-05-22 18:00
author:
  name: Dave Scott
  uri: http://dave.recoil.org/
  email: dave@recoil.org
subject: Building and packaging with dune and dune-release
permalink: packaging
---

## Packaging with dune and dune-release

This post describes the current state-of-the-art in building and releasing
MirageOS packages with
[dune](https://github.com/ocaml/dune) (to build)
and
[dune-release](https://github.com/samoht/dune-release) (to release).

Please note that some packages are still using `ocamlbuild` and `topkg`, install
`topkg-care` and replace `dune-release` with `topkg` below.

### Goals

We wish to

- make development and releasing of individual components as quick and as easy
  as possible
- use a similar structure across the MirageOS suite of components to make it
  easier for new people (and automated tools) to work across more than one
  component at a time

### The tools

We make heavy use of the following tools:

- [opam](https://github.com/ocaml/opam): defines a notion of a package, with versioned dependencies on other
  packages inside a package repository.
  We use this to ensure that we have a compatible set of component versions installed
  for our current project.
- [dune](https://github.com/ocaml/dune): a build tool (like `make`) which knows how to build OCaml code
  incrementally and really quickly.
- [dune-release](https://github.com/samoht/dune-release): a release tool which assists with tagging and uploading artefacts
  to github.

### Conventions

We adopt the following conventions:

- we prefix releases with `v` to easily distinguish concreate releases (`v1.2.3`) from release branches (`1.2`)
- we don't use opam's `depopts` to specify sub-libraries. Instead we create
  multiple `opam` packages via multiple `<name>.opam` files in the same repo.
  See [rgrinberg](http://rgrinberg.com/posts/optional-dependencies-considered-harmful/)'s
  post for a rationale
- we prefer to use the same name for both the `ocamlfind` package and the `opam` package. This is to avoid misunderstandings over whether you need to type `mirage-types.lwt` or `mirage-types-lwt` in the current context.
- we write `CHANGES.md` entries in the same style, to ensure they are parseable
  by `dune-release`
- we do not enable warnings as errors in the repo; instead we turn these on for
  local developer builds only. This is to prevent released versions from breaking
  when a future compiler version is released.

### Package structure

A MirageOS library should have

- `CHANGES.md`: containing a log of user-visible changes in each release.
  For example consider [mirage-tcpip CHANGES.md](https://github.com/mirage/mirage-tcpip/blob/v3.7.1/CHANGES.md):
  it has a markdown `###` prefix before each release version and the date in
  `(YYYY-MM-DD)` form.
- `LICENSE.md`: describing the conditions under which the code can be used
  (the MirageOS standard license is ISC).
  For example [mirage-tcpip LICENSE.md](https://github.com/mirage/mirage-tcpip/blob/v3.7.1/LICENSE.md).
- `README.md`: describing what the code is for and linking to examples, docs,
  continuous integration (CI) status. For example [mirage-tcpip.3.7.1](https://github.com/mirage/mirage-tcpip/blob/v3.7.1/README.md).
- one `<name>.opam` file per opam package defined in the repo.
  For example [mirage-block.1.2.0](https://github.com/mirage/mirage-block/blob/1.2.0/mirage-block.opam)
  and [mirage-block-lwt.1.2.0](https://github.com/mirage/mirage-block/blob/1.2.0/mirage-block-lwt.opam).
  These should have a github pages `doc:` link in order that `dune-release` can detect
  the upstream repo.
- `Makefile`: contains `dune` invocations.
  For example [mirage-block.1.2.0](https://github.com/mirage/mirage-block/blob/1.2.0/Makefile)
- one or more `dune` files: these describe how to build the libraries, executables
  and tests of your project.
  For example [mirage-block-unix.2.11.0/lib/dune](https://github.com/mirage/mirage-block-unix/blob/v2.11.0/lib/dune)
  links the main library against OCaml and C,
  while [mirage-block-unix.2.11.0/lib_test/dune](https://github.com/mirage/mirage-block-unix/blob/v2.11.0/lib_test/dune)
  defines 2 executables and associates one with an alias `runtest`, triggered by
  `make test` in the root.

### Developing changes

It should be sufficient to

- `git clone` the repo
- `opam install --deps-only <name>`: to install any required dependencies

and then

- `make`: to perform an incremental build
- `make test`: to compile and execute tests
- `dune utop`: to launch an interactive top-level

### Releasing changes

MirageOS releases are published via github. First log into your account and create
a GitHub API token if you haven't already. Store it in a file (e.g. `~/.config/dune/github.token`).
If on a multi-user machine, ensure the privileges are set to prevent other users
from reading it.

Before releasing anything it's a good idea to review the outstanding issues.
Perhaps some can be closed already? Maybe a `CHANGES.md` entry is missing?

When ready to go, create a branch from `master` and edit the `CHANGES.md` file
to list the interesting changes made since the last release. Make a pull request (PR) for this
update. The CI will run which is a useful final check that the code still builds
and the tests still pass.
(It's
ok to skip this if the CI was working fine a few moments ago when you merged
another PR). If you include `[ci skip]` in your commit message, the CI will not be run.

When the `CHANGES.md` PR is merged, pull it into your local `master` branch.

Read `dune-release help release` to have an overview of the full release workflow.
You need to have `odoc` installed to generate the documentation.

Type:

```
dune-release tag
```
-- dune-release will extract the latest version from the `CHANGES.md` file, perform
version substitutions and create a local tag.

Type:

```
dune-release distrib
```
-- dune-release will create a release tarball.

Install `odoc` and type:

```
dune-release publish --dry-run
```
-- dune-release will build the documentation (fix all the warnings).

Type:
```
dune-release publish
```
-- dune-release will push the tag, create a release and upload the release tarball.
It will also build the docs and push them online.

Type
```
dune-release opam pkg
dune-release opam submit
```

-- this will add new files in your opam-repository clone. `git commit` and push them to your fork on GitHub
and open a new pull-request.

You can simply write:

```
dune-release tag && dune-release
```
-- this will do the above steps (distrib, publish, opam pkg, opam submit).

