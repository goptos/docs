# Goptos

A fine grained reactive web framework in Go.

*Inspired by Leptos and SolidJS*

## Prerequisites

On a linux system the below prerequisites can be completed with the provided Makefile.

``` bash
make pre
```

Alternatively, install the following prerequisites manually.

**Go**

[Go: Download and install](https://go.dev/doc/install)

**Goptos CLI**

> If Go is not picking up the latest tagged version set `GONOPROXY` to `github.com/goptos` and try again.

``` bash
# Bypass go's proxy for goptos repos
export GONOPROXY=github.com/goptos

# Install goptos cli
go install github.com/goptos/goptos@latest
```

## Setup

Create a new directory for your project and use the Goptos CLI to initialise it.

``` bash
# Create a new directory for this project
mkdir myGoptosApp
cd myGoptosApp

# Initialise with the starter template
goptos init
```

## Build

Use the provided Makefile to build and serve your project.

``` bash
# Build and package (ready for publishing)
make build

# Build, package and start a local web server for development
make serve
```

> On Windows you will need to interpret the commands and run them manually.