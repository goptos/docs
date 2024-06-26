# Goptos

A fine-grained reactive web framework in Go.

*Inspired by Leptos and SolidJS*

## Prerequisites

On a linux system the below prerequisites can be completed with the provided Makefile.

``` bash
make pre
```

Alternatively, install the following prerequisites manually.

### Go

Follow the installation steps provided by the Go docs. Reference the Makefile for additional setup information.

[Go: Download and install](https://go.dev/doc/install)

### Goptos CLI

If Go is not picking up the latest tagged version of the Goptos CLI, set `GONOPROXY` to `github.com/goptos` and try again.

``` bash
# Bypass go's proxy for goptos repos
export GONOPROXY=github.com/goptos

# Install goptos cli
go install github.com/goptos/goptos@latest
```
## Usage

### Setup

Create a new directory for your project and use the `goptos init` to initialise it.

``` bash
# Create a new directory for this project
mkdir myGoptosApp
cd myGoptosApp

# Initialise with the starter template
goptos init
```

### Build

Use `goptos build` to build your project.

This will run `go generate` to generate view code from the view templates within each component, followed by `go build`.

``` bash
# Generate view code from templates and build
goptos build
```

> Setting the required environment variables for building to the wasm build target are handled by the Goptos CLI.

### Package

Use `goptos package` to bundle the needed wasm_exec glue code in with your built wasm binary.

``` bash
# Package ready for publishing
goptos package
```

> This will also copy in your index.html from the root of the project.

### Serve

Both the build and the package steps can be run together using `goptos serve` which will also start a local web server for development.

``` bash
# Build, package and start a local web server for development
goptos serve
```

> There is no watch feature at this curren time, so you will need to re-run `goptos serve` each time you make code changes.

## Contributing

Please refer to https://github.com/goptos/app
