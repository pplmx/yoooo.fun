---
categories:
    - golang
date: 2023-11-27T13:25:42Z
description: Here is a quick start for call cxx-static-lib in Go on Windows(MinGW)/Linux.
keywords:
    - golang
    - go
    - cgo
    - static
    - lib
    - cxx
    - g++
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
    - cxx
    - g++
    - mingw
    - windows
    - linux
title: Call cxx-static-lib in Go
---



# How to compile and call a CXX-static library in Golang

## touch some files first

```bash
# create a directory for this project
mkdir -p ./hi/lib; cd ./hi
touch ./main.go ./lib/hi.h ./lib/hi.cpp
```

## Create a header file: hi.h

In comparison with [c-static-lib](https://blog.caoyu.info/golang-call-c-static-lib.html), the header file here has some extra contents

> extern "C" { ... }

mainly to be compatible with C++ compilers.

```cpp
#ifndef HI_H
#define HI_H

#ifdef __cplusplus
extern "C" {
#endif

void print_hi();

#ifdef __cplusplus
}
#endif

#endif // HI_H

```

## Create a source file: hi.cpp

Different from [c-static-lib](https://blog.caoyu.info/golang-call-c-static-lib.html), the c++ source file must include the header file.

> #include "hi.h"

```cpp
#include <iostream>
#include "hi.h"

void print_hi() {
    std::cout << "hi, from C++\n";
}

```

## Create a main file: main.go

Compared to [c-static-lib](https://blog.caoyu.info/golang-call-cxx-static-lib.html), the LDFLAGS here has an extra `-l stdc++` to link the C++
standard library.

```go
package main

/*
#cgo CFLAGS: -I ${SRCDIR}/lib
#cgo LDFLAGS: -L ${SRCDIR}/lib -l hi -l stdc++
#include "hi.h"
*/
import "C"

func main() {
    C.print_hi()
}

```

## Compile the source file into a static library

```bash
g++ -c ./lib/hi.cpp -o ./lib/hi.o
ar rcs ./lib/libhi.a ./lib/hi.o
```

## Run

```bash
# go run directly
go run main.go

# or build and run
go build -o bin/hi main.go && ./bin/hi
```

## Summary

The key points are different from [call c-static-lib](https://blog.caoyu.info/golang-call-c-static-lib.html):

- use `extern "C" { ... }` in header file to be compatible with C++ compilers
- source file must include the header file
- add `-l stdc++` to `LDFLAGS`
- use `g++` to compile the source file into a static library

If you are not sure the static lib is `c` or `c++`,
your golang code can always use the `c++` way to call it, which is more compatible.

> Always add `-l stdc++` to `LDFLAGS`

## FYI

[Source Code](https://github.com/pplmx/cgo_demo)
