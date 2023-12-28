---
categories:
    - golang
date: 2023-11-24T10:29:42Z
description: Here is a quick start for call c-static-lib in Go on Windows(MinGW)/Linux.
keywords:
    - golang
    - go
    - cgo
    - static
    - lib
    - c
    - gcc
    - mingw
    - windows
    - linux
lastmod: 2023-11-27T13:25:42Z
tags:
    - golang
    - go
    - cgo
    - static
    - lib
    - c
    - gcc
    - mingw
    - windows
    - linux
title: Call c-static-lib in Go
---



# How to compile and call a C-static lib in Golang

## touch some files first

```bash
# create a directory for this project
mkdir -p ./hi/lib; cd ./hi
touch ./main.go ./lib/hi.h ./lib/hi.c
```

## Create a header file: hi.h

```c
#ifndef HI_H
#define HI_H

void print_hi();

#endif // HI_H

```

## Create a source file: hi.c

```c
#include <stdio.h>

void print_hi() {
    printf("hi, from C\n");
}

```

## Create a main file: main.go

```go
package main

/*
#cgo CFLAGS: -I ${SRCDIR}/lib
#cgo LDFLAGS: -L ${SRCDIR}/lib -l hi
#include "hi.h"
*/
import "C"

func main() {
	C.print_hi()
}

```

## Compile the source file into a static library

```bash
gcc -c ./lib/hi.c -o ./lib/hi.o
ar rcs ./lib/libhi.a ./lib/hi.o
```

## Run

```bash
# go run directly
go run main.go

# or build and run
go build -o bin/hi main.go && ./bin/hi
```

## FYI

[Source Code](https://github.com/pplmx/cgo_demo)
