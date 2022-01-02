# Go Modules

This page was created using the content of the post [Using Go Modules](https://go.dev/blog/using-go-modules), which contains more detailed information.

I repeated the exercises from the blog post and I created this page combining my own commands and the explanations of the blog posts. The resulting code can be found in directory [hello](hello).

## Create a Go module

Let’s create a new module.

Create a new, empty directory somewhere outside `$GOPATH/src`, then cd into that directory. 

```
$ mkdir hello
$ cd hello
```

Create a new source file, [hello.go](hello/hello.go):

```go
package hello

func Hello() string {
    return "Hello, world."
}
```

Let’s write a test, too, in [hello_test.go](hello/hello_test.go):

```go
package hello

import "testing"

func TestHello(t *testing.T) {
    want := "Hello, world."
    if got := Hello(); got != want {
        t.Errorf("Hello() = %q, want %q", got, want)
    }
}
```

At this point, the directory contains a package, but not a module, because there is no `go.mod file`. If we were working in [hello](hello) and ran `go test` now, we’d see:

```
$ go test
go: cannot find main module, but found .git/config in /home/joseba/Develop/labs-golang
        to create a module there, run:
        cd ../.. && go mod init
```

Let’s make the current directory the root of a module by using `go mod init` and then try go test again:

```
$ go mod init example.com/hello
go: creating new go.mod: module example.com/hello
go: to add module requirements and sums:
        go mod tidy
```

```
$ go test
PASS
ok      example.com/hello       0.002s
```

Congratulations! You’ve written and tested your first module.

The go mod init command wrote a [go.mod](hello/go.mod) file:

```go
module example.com/hello

go 1.17
```

The [go.mod](hello/go.mod) file only appears in the root of the module. Packages in subdirectories have import paths consisting of the module path plus the path to the subdirectory. For example, if we created a subdirectory `world`, we would not need to (nor want to) run `go mod init` there. The package would automatically be recognized as part of the `example.com/hello` module, with import path `example.com/hello/world`.


## Adding a dependency

The primary motivation for Go modules was to improve the experience of using (that is, adding a dependency on) code written by other developers.

Let’s update our [hello.go](hello/hello.go) to import `rsc.io/quote` and use it to implement Hello:

```go
package hello

import "rsc.io/quote"

func Hello() string {
    return quote.Hello()
}
```

Now let’s run the test again:

```
$ go test
hello.go:3:8: no required module provides package rsc.io/quote; to add it:
        go get rsc.io/quote
```

To install `rsc.io/quote`:

```
$ go get rsc.io/quote
```

One-liner to install all the dependencies of a project

```
$ go get .
```

Run  `go test` again:

```
$ go test
PASS
ok      example.com/hello       0.002s
```

Adding one direct dependency often brings in other indirect dependencies too. The command go list -m all lists the current module and all its dependencies:

```
$ go list -m all
example.com/hello
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
rsc.io/quote v1.5.2
rsc.io/sampler v1.3.0
```

In the go list output, the current module, also known as the main module, is always the first line, followed by dependencies sorted by module path.

The golang.org/x/text version v0.0.0-20170915032832-14c0d48ead0c is an example of a pseudo-version, which is the go command’s version syntax for a specific untagged commit.

In addition to go.mod, the go command maintains a file named go.sum containing the expected cryptographic hashes of the content of specific module versions:

```
$ cat go.sum
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c h1:qgOY6WgZOaTkIIMiVjBQcw93ERBE4m30iBm00nkL0i8=
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c/go.mod h1:NqM8EUOU14njkJ3fqMW+pc6Ldnwhi/IjpwHt7yyuwOQ=
rsc.io/quote v1.5.2 h1:w5fcysjrx7yqtD/aO+QwRjYZOKnaM9Uh2b40tElTs3Y=
rsc.io/quote v1.5.2/go.mod h1:LzX7hefJvL54yjefDEDHNONDjII0t9xZLPXsUe+TKr0=
rsc.io/sampler v1.3.0 h1:7uVkIFmeBqHfdjD+gZwtXXI+RODJ2Wc4O7MPEh/QiW4=
rsc.io/sampler v1.3.0/go.mod h1:T1hPZKmBbMNahiBKFy5HrXp6adAjACjK9JXDnKaTXpA=
```

The go command uses the go.sum file to ensure that future downloads of these modules retrieve the same bits as the first download, to ensure the modules your project depends on do not change unexpectedly, whether for malicious, accidental, or other reasons. Both go.mod and go.sum should be checked into version control.

## Upgrading dependencies

With Go modules, versions are referenced with semantic version tags. A semantic version has three parts: major, minor, and patch. For example, for v0.1.2, the major version is 0, the minor version is 1, and the patch version is 2. Let’s walk through a couple minor version upgrades. In the next section, we’ll consider a major version upgrade.

From the output of go list -m all, we can see we’re using an untagged version of `golang.org/x/text`. Let’s upgrade to the latest tagged version and test that everything still works:

```
$ go get golang.org/x/text
go get: upgraded golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c => v0.3.7
```
Make test:

```
$ go test
PASS
ok      example.com/hello       0.003s
```

Woohoo! Everything passes. Let’s take another look at go list -m all and the go.mod file:

```
$ go list -m all
example.com/hello
golang.org/x/text v0.3.7
golang.org/x/tools v0.0.0-20180917221912-90fa682c2a6e
rsc.io/quote v1.5.2
rsc.io/sampler v1.3.0
```

```
$ cat go.mod
module example.com/hello

go 1.17

require rsc.io/quote v1.5.2

require (
        golang.org/x/text v0.3.7 // indirect
        rsc.io/sampler v1.3.0 // indirect
)

```

The `golang.org/x/text` package has been upgraded to the latest tagged version (v0.3.0). The go.mod file has been updated to specify `v0.3.0` too. The indirect comment indicates a dependency is not used directly by this module, only indirectly by other module dependencies. See go help modules for details.

Now let’s try upgrading the `rsc.io/sampler` minor version. Start the same way, by running go get and running tests:

```
$ go get rsc.io/sampler
go: downloading rsc.io/sampler v1.99.99
go get: upgraded rsc.io/sampler v1.3.0 => v1.99.99
```

```
$ go test
--- FAIL: TestHello (0.00s)
    hello_test.go:8: Hello() = "99 bottles of beer on the wall, 99 bottles of beer, ...", want "Hello, world."
FAIL
exit status 1
FAIL	example.com/hello	0.002s
```

Uh, oh! The test failure shows that the latest version of `rsc.io/sampler` is incompatible with our usage. Let’s list the available tagged versions of that module:

```
$ go list -m -versions rsc.io/sampler
rsc.io/sampler v1.0.0 v1.2.0 v1.2.1 v1.3.0 v1.3.1 v1.99.99
```

We had been using v1.3.0; v1.99.99 is clearly no good. Maybe we can try using v1.3.1 instead:

```
$ go get rsc.io/sampler@v1.3.1
go: downloading rsc.io/sampler v1.3.1
go get: downgraded rsc.io/sampler v1.99.99 => v1.3.1
```

```
$ go test
PASS
ok  	example.com/hello	0.004s
```

Note the explicit @v1.3.1 in the go get argument. In general each argument passed to go get can take an explicit version; the default is @latest, which resolves to the latest version as defined earlier.

## Adding a dependency on a new major version

Let’s add a new function to our package: func Proverb returns a Go concurrency proverb, by calling quote.Concurrency, which is provided by the module `rsc.io/quote/v3`. First we update [hello.go](hello/hello.go) to add the new function:


```go
package hello

import (
	"rsc.io/quote"
	quoteV3 "rsc.io/quote/v3"
)

func Hello() string {
	return quote.Hello()
}

func Proverb() string {
	return quoteV3.Concurrency()
}
```

Then we add a test to [hello_test.go](hello/hello_test.go):


```go
package hello

import "testing"

func TestHello(t *testing.T) {
	want := "Hello, world."
	if got := Hello(); got != want {
		t.Errorf("Hello() = %q, want %q", got, want)
	}
}

func TestProverb(t *testing.T) {
	want := "Concurrency is not parallelism."
	if got := Proverb(); got != want {
		t.Errorf("Proverb() = %q, want %q", got, want)
	}
}
```

Download the dependencies:

```
$ go get .
```

Then we can test our code:

```
$ go test
PASS
ok      example.com/hello       0.003s
```

Note that our module now depends on both rsc.io/quote and rsc.io/quote/v3:

```
$ go list -m rsc.io/q...
rsc.io/quote v1.5.2
rsc.io/quote/v3 v3.1.0
```

Each different major version (v1, v2, and so on) of a Go module uses a different module path: starting at v2, the path must end in the major version. In the example, v3 of rsc.io/quote is no longer rsc.io/quote: instead, it is identified by the module path rsc.io/quote/v3. This convention is called semantic import versioning, and it gives incompatible packages (those with different major versions) different names. In contrast, v1.6.0 of rsc.io/quote should be backwards-compatible with v1.5.2, so it reuses the name rsc.io/quote. (In the previous section, rsc.io/sampler v1.99.99 should have been backwards-compatible with rsc.io/sampler v1.3.0, but bugs or incorrect client assumptions about module behavior can both happen.)

The go command allows a build to include at most one version of any particular module path, meaning at most one of each major version: one rsc.io/quote, one rsc.io/quote/v2, one rsc.io/quote/v3, and so on. This gives module authors a clear rule about possible duplication of a single module path: it is impossible for a program to build with both rsc.io/quote v1.5.2 and rsc.io/quote v1.6.0. At the same time, allowing different major versions of a module (because they have different paths) gives module consumers the ability to upgrade to a new major version incrementally. In this example, we wanted to use quote.Concurrency from rsc/quote/v3 v3.1.0 but are not yet ready to migrate our uses of rsc.io/quote v1.5.2. The ability to migrate incrementally is especially important in a large program or codebase.

## Upgrading a dependency to a new major version

Let’s complete our conversion from using rsc.io/quote to using only rsc.io/quote/v3. Because of the major version change, we should expect that some APIs may have been removed, renamed, or otherwise changed in incompatible ways. Reading the docs, we can see that Hello has become HelloV3:

```
$ go doc rsc.io/quote/v3
package quote // import "rsc.io/quote/v3"

Package quote collects pithy sayings.

func Concurrency() string
func GlassV3() string
func GoV3() string
func HelloV3() string
func OptV3() string
```

We can update our use of `quote.Hello()` in [hello.go](hello/hello.go) to use `quoteV3.HelloV3()`:

```go
package hello

import quoteV3 "rsc.io/quote/v3"

func Hello() string {
	return quoteV3.HelloV3()
}

func Proverb() string {
	return quoteV3.Concurrency()
}
```

And then at this point, there’s no need for the renamed import anymore, so we can undo that:


```go
package hello

import "rsc.io/quote/v3"

func Hello() string {
	return quote.HelloV3()
}

func Proverb() string {
	return quote.Concurrency()
}
```

Let’s re-run the tests to make sure everything is working:


```
$ go test
PASS
ok      example.com/hello       0.005s
```

## Removing unused dependencies

We’ve removed all our uses of rsc.io/quote, but it still shows up in `go list -m all` and in our [go.mod](hello/go.mod) file:

```
$ go list -m all
example.com/hello
golang.org/x/text v0.3.7
golang.org/x/tools v0.0.0-20180917221912-90fa682c2a6e
rsc.io/quote v1.5.2
rsc.io/quote/v3 v3.1.0
rsc.io/sampler v1.3.1
```

```go
module example.com/hello

go 1.17

require (
        rsc.io/quote v1.5.2
        rsc.io/quote/v3 v3.1.0
)

require (
        golang.org/x/text v0.3.7 // indirect
        rsc.io/sampler v1.3.1 // indirect
)

```

Why? Because building a single package, like with go build or go test, can easily tell when something is missing and needs to be added, but not when something can safely be removed. Removing a dependency can only be done after checking all packages in a module, and all possible build tag combinations for those packages. An ordinary build command does not load this information, and so it cannot safely remove dependencies.

The go mod tidy command cleans up these unused dependencies:


```
$ go mod tidy
```

Module list after the command:

```
$ go list -m all
example.com/hello
golang.org/x/text v0.3.7
golang.org/x/tools v0.0.0-20180917221912-90fa682c2a6e
rsc.io/quote/v3 v3.1.0
rsc.io/sampler v1.3.1
```

File [go.mod](hello/go.mod) after the command:

```go
module example.com/hello

go 1.17

require rsc.io/quote/v3 v3.1.0

require (
        golang.org/x/text v0.3.7 // indirect
        rsc.io/sampler v1.3.1 // indirect
)

```

## Conclusion

Go modules are the future of dependency management in Go. Module functionality is now available in all supported Go versions (that is, in Go 1.11 and Go 1.12).

This post introduced these workflows using Go modules:

- `go mod init` creates a new module, initializing the go.mod file that describes it.
- `go build`, `go test`, and other package-building commands add new dependencies to `go.mod` as needed.
- `go list -m all` prints the current module’s dependencies.
- `go get` changes the required version of a dependency (or adds a new dependency).
- `go mod tidy` removes unused dependencies.

We encourage you to start using modules in your local development and to add `go.mod` and `go.sum` files to your projects. To provide feedback and help shape the future of dependency management in Go, please send us bug reports or experience reports.

Thanks for all your feedback and help improving modules.
