# How to test private functions when performing blackbox testing in Go

## Introduction

There are two main strategies to perform unit testing so let's start with some basic terminology. Let's imagine we have a file named `foo.go`

```go
// foo.go

package myfoo

func Foo(i int) int {
    return i + boo(i)
}

func boo(i int) int {
    if i == 2 {
        return 2
    }
    return i
}
```

* Black-box Testing: Use package `myfoo_test` for your unit tests files, which will ensure you're only using the exported identifiers. That is `Foo`.

* White-box Testing: Use package `myfoo` so that you have access to the non-exported identifiers, like `boo`. Good for unit tests that require access to non-exported variables, functions, and methods.

In theory when it comes to unit testing is a good practice to test only public methods. Private methods should be tested through the public methods that they are called. Sometimes though this can become complicated and hard. For example if `boo` has multiple test cases I need to call `Foo` for each case providing everything that it needs to achieve my goal.

In these cases I wish I could test directly `boo` but without using White-box Testing. Well there is a way.

## Test your private functions using Black-box Testing

Let's create an `export_test.go` file that will export the private functions of `myfoo`. The file needs to be part of `myfoo` but because is a `_test` file its content will not be compiled in the binary. So we are just exposing these functions in `myfoo_test.go`

```go
// export_test.go
var (
    Boo = boo
)
```

Now we can write a test for `boo` by calling `Boo` in `myfoo_test.go`.