# Stellar Go 
[![Build Status](https://travis-ci.org/stellar/go.svg?branch=master)](https://travis-ci.org/stellar/go) 
[![GoDoc](https://godoc.org/github.com/stellar/go?status.svg)](https://godoc.org/github.com/stellar/go)
[![Go Report Card](https://goreportcard.com/badge/github.com/stellar/go)](https://goreportcard.com/report/github.com/stellar/go)

This repo is the home for all of the public go code produced by SDF.  In addition to various tools and services, this repository is the SDK from which you may develop your own applications that integrate with the stellar network.

## Getting started

To develop for Horizon on **macOS 12.13**, install [`brew`](brew.sh) and follow the steps below in your command line:

```sh
# Install the database dependencies and the go framework.
brew install postgresql redis go dep
# Create a go "workspace" where go code related to this repository will be stored.
# The $GOPATH can be any directory, but the location of this repository within the
# $GOPATH needs to be located in a go-conventional place. 
export GOPATH="$(pwd)/stellar-gopath"
mkdir -pv "$GOPATH/src/github.com/stellar"
git clone https://github.com/stellar/go.git $GOPATH/src/github.com/stellar/go
cd "$GOPATH/src/github.com/stellar/go"
# Download and install all the dependencies for this repository
dep ensure -v
```

Your `GOPATH` variable will need to be exported with `export GOPATH=path/to/directory/chosen/above` in every shell. There is no consensus whether a `GOPATH` should exist for an entire single machine, user or logical group of projects. Since the group of projects convention is used here, think of this `GOPATH` variable as the equivalent `source venv/bin/activate`, `rbenv ...`, or `./gradlew ...` "set up" command for a Go environment for this repository's development. It would not necessarily be appropriate to export this `GOPATH` inside a `.bashrc` or `.zshrc` as documented elsewhere.

JetBrain's **GoLand** is a well-supported commercial IDE for Go. Open the `$GOPATH/src/github.com/stellar/go` directory with it. Then in the application's Preferences, open the Go configuration tree, and:

 1. Visit GOROOT, and set the GOROOT to the `brew` installed location. The dropdown will locate it automatically.
 2. Visit GOPATH, and add a Project GOPATH pointing to the directory of your exported GOPATH variable.
 3. Visit dep, and check Enable dep integration. The dropdown will locate it automatically.
 
Some tests require the databases to be properly running the background in order to run.

The majority of tests require postgres. First create a data directory using a specific `pg_ctl` command: `pg_ctl initdb -D $GOPATH/data`. Note, the data directory can be anywhere, and there are no conventions in Go for where this directory is specified. Then, to start it as a daemon:

```sh
# From any directory
pg_ctl start -D $GOPATH/data
```

To stop the postgres daemon, use:

```sh
# From any directory
pg_ctl stop -D $GOPATH/data
```

Some tests, like rate limiting and connecting to a mysql database (not typical usage) require those dependencies running when the test runs.

To run redis as a daemon, simply start it in your local directory with a routine POSIX backgrounding pattern:

```sh
# From any directory
redis-server >/dev/null 2>&1 &
```

This will show a PID value you can later use to stop it. For example, if the PID value was 19000, stop it with:

```sh
kill -2 19000
```

On macOS 12.13 installed from `brew`, mysql will currently fail to link its command line binaries into the path. Additionally, even when running, the default configuration of mysql does not support the root user login the test in this repository requires.

To run all tests except the mysql test, we'll omit it using the following `go test` command:

```sh
# From the repository directory
cd "$GOPATH/src/github.com/stellar/go"
go test -run "\(\(\?\!mysql\).\)*" ./... -p 8
```

To build the underlying executables:

```
# From the repository directory
CGO_ENABLED=1 go run ./support/scripts/build_release_artifacts/main.go -os darwin
```

This will place distributions in the `dist/` directory. Extract one, like Horizon, and execute it:

```sh
cd dist
tar -xvf horizon-snapshot-darwin-amd64.tar.gz
./horizon-snapshot-darwin-amd64/horizon --help
```

These command line arguments correspond to the `main.go` entry point of the respective binary's source tree. For horizon, this file would be located at `services/horizon/main.go`.

## Dependencies

This repository depends upon a [number of external dependencies](./Gopkg.lock), and we use [dep](https://golang.github.io/dep/) to manage them.  Dep is used to populate the [vendor directory](https://golang.github.io/dep/docs/ensure-mechanics.html), ensuring that builds are reproducible even as upstream dependencies are changed. Please see the [dep](https://golang.github.io/dep/) website for installation instructions.

You can use dep yourself in your project and add stellar go as a vendor'd dependency, or you can just drop this repos as `$GOPATH/src/github.com/stellar/go` to import it the canonical way (you still need to run `dep ensure -v`).

When creating this project, we had to decide whether or not we committed our external dependencies to the repo.  We decided that we would not, by default, do so.  This lets us avoid the diff churn associated with updating dependencies while allowing an acceptable path to get reproducible builds.  To do so, simply install dep and run `dep ensure -v` in your checkout of the code.  We realize this is a judgement call; Please feel free to open an issue if you would like to make a case that we change this policy.


## Directory Layout

In addition to the other top-level packages, there are a few special directories that contain specific types of packages:

* **clients** contains packages that provide client packages to the various Stellar services.
* **exp** contains experimental packages.  Use at your own risk.
* **handlers** contains packages that provide pluggable implementors of `http.Handler` that make it easier to incorporate portions of the Stellar protocol into your own http server. 
* **support** contains packages that are not intended for consumption outside of Stellar's other packages.  Packages that provide common infrastructure for use in our services and tools should go here, such as `db` or `log`. 
* **support/scripts** contains single-file go programs and bash scripts used to support the development of this repo. 
* **services** contains packages that compile to applications that are long-running processes (such as API servers).
* **tools** contains packages that compile to command line applications.

Each of these directories have their own README file that explain further the nature of their contents.

### Other packages

In addition to the packages described above, this repository contains various packages related to working with the Stellar network from a go program.  It's recommended that you use [godoc](https://godoc.org/github.com/stellar/go#pkg-subdirectories) to browse the documentation for each.


## Package source layout

While much of the code in individual packages is organized based upon different developers' personal preferences, many of the packages follow a simple convention for organizing the declarations inside of a package that aim to aid in your ability to find code.

In each package, there may be one or more of a set of common files:

- *main.go*: Every package should have a `main.go` file.  This file contains the package documentation (unless a separate `doc.go` file is used), _all_ of the exported vars, consts, types and funcs for the package. 
- *internal.go*:  This file should contain unexported vars, consts, types, and funcs.  Conceptually, it should be considered the private counterpart to the `main.go` file of a package
- *errors.go*: This file should contains declarations (both types and vars) for errors that are used by the package.
- *example_test.go*: This file should contains example tests, as described at https://blog.golang.org/examples.

In addition to the above files, a package often has files that contains code that is specific to one declared type.  This file uses the snake case form of the type name (for example `loggly_hook.go` would correspond to the type `LogglyHook`).  This file should contain method declarations, interface implementation assertions and any other declarations that are tied solely to to that type.

Each non-test file can have a test counterpart like normal, whose name ends with `_test.go`.  The common files described above also have their own test counterparts... for example `internal_test.go` should contains tests that test unexported behavior and more commonly test helpers that are unexported.

Generally, file contents are sorted by exported/unexported, then declaration type  (ordered as consts, vars, types, then funcs), then finally alphabetically.

### Test helpers

Often, we provide test packages that aid in the creation of tests that interact with our other packages.  For example, the `support/db` package has the `support/db/dbtest` package underneath it that contains elements that make it easier to test code that accesses a SQL database.  We've found that this pattern of having a separate test package maximizes flexibility and simplifies package dependencies.


## Coding conventions

- Always document exported package elements: vars, consts, funcs, types, etc.
- Tests are better than no tests.
