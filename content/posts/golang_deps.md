---
title: "Dependencies in Go"
date: 2018-01-17T10:48:00Z
tags: ["go"]
draft: false
---

# Intro

As I journey into Go land, one topic that quickly comes to the table is "How to handle dependencies?"

This post is my attempt at a birds-eye view on the topic. I hope newcomers to the language like me find useful.

## Basic

When starting a new go project, the most basic version is simply to

```
mkdir project
cd project
<edit> main.go
```

on the editor of your choice put

```go
package main

func main() {

}

```

and run

```
go run main.go
```

This will compile and run a program that does absolutely nothing.

## Standard library

Now, to do anything, we need, at least, the standard library. 

To that end, `main.go` turns into something like

```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("Hello!")
}
```
The Go standard lib is fairly complete but, obviously, it will not always have everything we need.
For that we'll need external dependencies.

## External dependencies


This is done via the standard `go get` tool. [link](https://golang.org/cmd/go/#hdr-Download_and_install_packages_and_dependencies)

Something like
`go get github.com/dustin/go-humanize`
will fetch the code to `$GOPATH/src` and allow you to do
```go
import(
    "github.com/dustin/go-humanize"
)

func main() {
    fmt.Printf("That file is %s.", humanize.Bytes(82854982)) // That file is 83 MB.
}
```
This works for simple tests and trials but has two main problems for a production setup:

  - It forces that, for each package, only one version can be used
  - It uses the latest commit from the default branch when compiling/installing.

## Dependency management

### vendor/

Go 1.5 started to fix these problems with the introduction of the `vendor` directory.
First behind an experimental vendoring flag, GO15VENDOREXPERIMENT and from Go 1.7 as a standard feature.

When Go looks up a package referenced in the source, it will look in the `vendor` directory, then look in the `GOPATH`, then look in the `GOROOT` (where standard library packages live).

This allows us to have specific versions of dependent packages per project and, as such, solves the first problem.

To solve the second problem we need version management...

### Dep

Go currently has no official dependency management tool but it's getting close. 
The [dep](https://github.com/golang/dep) tool looks like it will be part of the standard distribution.

Let's try it by installing and making it available on the `$PATH`.

Next, type `dep init` from the root directory of the project.

This will generate two files: `Gopkg.toml` and `Gopkg.lock`.

The `.toml` is the configuration file and can be edited by hand. The `.lock` is sort of a current status of the intent expressed by the `.toml` file and is updated by `dep`. For a more in-depth tutorial check out the [documentation](https://github.com/golang/dep/blob/master/docs/Gopkg.toml.md).

For this guide, the configuration block that interests us is `constraint`.

Edit `Gopkg.toml` and add the following

```
[[constraint]]
  name = "github.com/dustin/go-humanize"
  # Only one of "branch", "version" or "revision" can be specified.
  # version = "1.0.0"
  branch = "master"
  # revision = "abc123"
```

next run `dep ensure`.

After that, you'll have the `vendor/` directory with the repository contents of the dependency you declared.
The file `Gopkg.lock` was also updated to reflect the added dependency.

You can also run `dep status` to see details about the dependencies.

```
$ dep status
PROJECT                        CONSTRAINT     VERSION        REVISION  LATEST   PKGS USED
github.com/dustin/go-humanize  branch master  branch master  bb3d318   bb3d318  1
```

The tool is very young and really slow on some operations but I'll assume it will be in a much better shape when it's finally included in the standard distribution.

# Outro

Coming from a Java ecosystem where tools like Maven or Gradle are basically mandatory, I find this incremental approach to dependency management very refreshing.

Additionally, the fact that the Go ecosystem did not conflate _"dependency management"_ and _"build tool"_ like the mentioned tools from Java land is also a very good sign in my view.

