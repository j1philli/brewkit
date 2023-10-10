![pkgx](https://pkgx.dev/banner.png)

# BrewKit

BrewKit is build infrastructure for `pkgx`.

```sh
$ env +brewkit
$ pkg build node
```

If you are inside a pantry then BrewKit will figure out what packages you are
editing and build them.

```sh
$ cd pantry
$ dev
$ pkg edit openssl
# make some edits…
$ pkg build
brewkit: building openssl.org
```

You can build for Linux (via Docker) using `-L`, e.g.:

```sh
pkg -L build
```

## Some Details

* The `pkg build` environment ensures you have a c/c++ compiler and `make`
  * We do this to ensure builds on Mac and Linux have a consistent base layer
    of tooling
  * We also provide shims so use of tools like `pkg-config` just work
  * We shim `sed` since Mac comes with BSD sed and Linux typically comes with
    GNU sed and they are subtly different
* The `pkg test` environment automatically adds pkgs when called
  * We emit a warning for this since sometimes these deps should be  explicit

## Outside a Pantry Checkout

Outside a pantry checkout we operate against your `pkgx` installation
(which defaults to `~/.pkgx`). Builds occur in a temporary directory rather
than local to your pantry checkout.

```sh
env "$(pkgx +brewkit)" pkg build zlib.net
```


## Additions

This repo is for tooling built on top of the `pkgx` primitives with the purpose
of generalized building and testing of open source packages.

If you have an idea for an addition open a [discussion]!


### Stuff That Needs to be Added

Getting the `rpath` out of a macOS binary:

```sh
lsrpath() {
    otool -l "$@" |
    awk '
        /^[^ ]/ {f = 0}
        $2 == "LC_RPATH" && $1 == "cmd" {f = 1}
        f && gsub(/^ *path | \(offset [0-9]+\)$/, "") == 2
    '
}
```

This should be added to a `pkg doctor` type thing I reckon. E.g.
`pkg doctor zlib.net -Q:rpath`.


### Hacking on brewkit

In a pantry clone, if you do `../brewkit/bin/pkg build` for example your local
brewkit will be used rather than that which is installed.

&nbsp;



# Tasks

## Bump

Priority is one of `major|minor|patch|prerelease|VERSION`

Inputs: PRIORITY

```sh
./scripts/publish-release.sh $PRIORITY
```


## Shellcheck

```sh
for x in bin/*; do
  if file $x | grep 'shell script'; then
    pkgx shellcheck --shell=dash --severity=warning $x
  fi
done

pkgx shellcheck --shell=dash --severity=warning **/*.sh
```


[discussion]: https://github.com/orgs/pkgxdev/discussions
