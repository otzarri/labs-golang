# Tutorial: Fundamentals of the Go language

This is one of the tutorials of the Go documenation. This tutorial is composed by different related documentation topic, each one in a webpage.

In this tutorial you'll create two modules. The first is a library which is intended to be imported by other libraries or applications. The second is a caller application which will use the first.

This tutorial's sequence includes seven brief topics that each illustrate a different part of the language.

| Topic                                                                                                       | Description                                                           |
|-------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------|
| [Part 1 - Create a module](https://go.dev/doc/tutorial/create-module)                                       | Write a small module with functions you can call from another module. |
| [Part 2 - Call your code from another module](https://go.dev/doc/tutorial/call-module-code.html)            | Import and use your new module.                                       |
| [Part 3 - Return and handle an error](https://go.dev/doc/tutorial/handle-errors.html)                       | Add simple error handling.                                            |
| [Part 4 - Return a random greeting](https://go.dev/doc/tutorial/random-greeting.html)                       | Handle data in slices (Go's dynamically-sized arrays).                |
| [Part 5 - Return greetings for multiple people](https://go.dev/doc/tutorial/greetings-multiple-people.html) | Store key/value pairs in a map.                                       |
| [Part 6 - Add a test](https://go.dev/doc/tutorial/add-a-test.html)                                          | Use Go's built-in unit testing features to test your code.            |
| [Part 7 - Compile and install the application](https://go.dev/doc/tutorial/compile-install.html)            | Compile and install your code locally.                                |

## Part 1 - Create a module

Source page: https://go.dev/doc/tutorial/create-module

Start by creating a Go module. In a module, you collect one or more related packages for a discrete and useful set of functions. For example, you might create a module with packages that have functions for doing financial analysis so that others writing financial applications can use your work. For more about developing modules, see Developing and publishing modules.

Go code is grouped into packages, and packages are grouped into modules. Your module specifies dependencies needed to run your code, including the Go version and the set of other modules it requires.

As you add or improve functionality in your module, you publish new versions of the module. Developers writing code that calls functions in your module can import the module's updated packages and test with the new version before putting it into production use.

Open a command prompt and cd to your home directory.

```
$ cd
```

Create a greetings directory for your Go module source code.

```
$ mkdir greetings
$ cd greetings
```

Start your module using the go mod init command.

Run the go mod init command, giving it your module path -- here, use example.com/greetings. If you publish a module, this must be a path from which your module can be downloaded by Go tools. That would be your code's repository.

For more on naming your module with a module path, see Managing dependencies.

```
$ go mod init example.com/greetings
go: creating new go.mod: module example.com/greetings
```

The go mod init command creates a go.mod file to track your code's dependencies. So far, the file includes only the name of your module and the Go version your code supports. But as you add dependencies, the go.mod file will list the versions your code depends on. This keeps builds reproducible and gives you direct control over which module versions to use.

In your text editor, create a file in which to write your code and call it greetings.go.

Paste the following code into your greetings.go file and save the file.


```go
package greetings

import "fmt"

// Hello returns a greeting for the named person.
func Hello(name string) string {
    // Return a greeting that embeds the name in a message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message
}
```

This is the first code for your module. It returns a greeting to any caller that asks for one. You'll write code that calls this function in the next step.

In this code, you:
- Declare a greetings package to collect related functions.
- Implement a Hello function to return the greeting: This function takes a name parameter whose type is string. The function also returns a string. In Go, a function whose name starts with a capital letter can be called by a function not in the same package. This is known in Go as an exported name. For more about exported names, see [Exported names](https://go.dev/tour/basics/3) in the Go tour.
- Declare a message variable to hold your greeting: In Go, the `:=` operator is a shortcut for declaring and initializing a variable in one line (Go uses the value on the right to determine the variable's type). Taking the long way, you might have written this as:

```
        var message string
        message = fmt.Sprintf("Hi, %v. Welcome!", name)
```

- Use the `fmt` package's `Sprintf` function to create a greeting message. The first argument is a format string, and Sprintf substitutes the name parameter's value for the `%v` format verb. Inserting the value of the name parameter completes the greeting text.
- Return the formatted greeting text to the caller.

In the next step, you'll call this function from another module. 

## Part 2 - Call your code from another module

In the previous section, you created a greetings module. In this section, you'll write code to make calls to the Hello function in the module you just wrote. You'll write code you can execute as an application, and which calls code in the greetings module.

1. Create a hello directory for your Go module source code. This is where you'll write your caller.

After you create this directory, you should have both a hello and a greetings directory at the same level in the hierarchy, like so:

```
<home>/
 |-- greetings/
 |-- hello/
```

For example, if your command prompt is in the greetings directory, you could use the following commands:

```
cd ..
mkdir hello
cd hello
```

2. Enable dependency tracking for the code you're about to write.

To enable dependency tracking for your code, run the go mod init command, giving it the name of the module your code will be in.

For the purposes of this tutorial, use `example.com/hello` for the module path.

```
$ go mod init example.com/hello
go: creating new go.mod: module example.com/hello
```

3. In your text editor, in the [hello](hello) directory, create a file in which to write your code and call it [hello.go](hello/hello.go).

4. Write code to call the `Hello` function, then print the function's return value.

To do that, paste the following code into [hello/hello.go](hello/hello.go).

```go
package main

import (
    "fmt"

    "example.com/greetings"
)

func main() {
    // Get a greeting message and print it.
    message := greetings.Hello("Gladys")
    fmt.Println(message)
}
```

In this code, you:

- Declare a main package. In Go, code executed as an application must be in a main package.
- Import two packages: example.com/greetings and the fmt package. This gives your code access to functions in those packages. Importing example.com/greetings (the package contained in the module you created earlier) gives you access to the Hello function. You also import fmt, with functions for handling input and output text (such as printing text to the console).
- Get a greeting by calling the greetings package’s Hello function.

5. Edit the example.com/hello module to use your local example.com/greetings module. 

For production use, you’d publish the example.com/greetings module from its repository (with a module path that reflected its published location), where Go tools could find it to download it. For now, because you haven't published the module yet, you need to adapt the example.com/hello module so it can find the example.com/greetings code on your local file system.

To do that, use the go mod edit command to edit the example.com/hello module to redirect Go tools from its module path (where the module isn't) to the local directory (where it is).

From the command prompt in the hello directory, run the following command:

```
$ go mod edit -replace example.com/greetings=../greetings
```

The command specifies that example.com/greetings should be replaced with ../greetings for the purpose of locating the dependency. After you run the command, the go.mod file in the hello directory should include a replace directive:

```
$ cat go.mod
module example.com/hello

go 1.17

replace example.com/greetings => ../greetings
```

From the command prompt in the hello directory, run the go mod tidy command to synchronize the example.com/hello module's dependencies, adding those required by the code, but not yet tracked in the module.

```
$ go mod tidy
go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000

```

After the command completes, the example.com/hello module's go.mod file should look like this:

```
$ cat go.mod
module example.com/hello

go 1.17

replace example.com/greetings => ../greetings

require example.com/greetings v0.0.0-00010101000000-000000000000
```

The command found the local code in the greetings directory, then added a require directive to specify that `example.com/hello` requires `example.com/greetings`. You created this dependency when you imported the greetings package in hello.go.

The number following the module path is a pseudo-version number -- a generated number used in place of a semantic version number (which the module doesn't have yet).

To reference a published module, a [go.mod](hello/go.mod) file would typically omit the replace directive and use a require directive with a tagged version number at the end.

```go
require example.com/greetings v1.1.0
```

For more on version numbers, see [Module version numbering](https://go.dev/doc/modules/version-numbers).

6. At the command prompt in the hello directory, run your code to confirm that it works.

```
$ go run .
Hi, Gladys. Welcome!
```

Congrats! You've written two functioning modules.

In the next topic, you'll add some error handling.

## Return and handle an error

Handling errors is an essential feature of solid code. In this section, you'll add a bit of code to return an error from the greetings module, then handle it in the caller.

In [greetings/greetings.go](greetings/greetings.go), add the code highlighted below.

There's no sense sending a greeting back if you don't know who to greet. Return an error to the caller if the name is empty. Copy the following code into greetings.go and save the file.

```go
package greetings

import (
    "errors"
    "fmt"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return "", errors.New("empty name")
    }

    // If a name was received, return a value that embeds the name
    // in a greeting message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message, nil
}
```

In this code, you:

- Change the function so that it returns two values: a `string` and an `error`. Your caller will check the second value to see if an error occurred. (Any Go function can return multiple values. For more, see [Effective Go](https://go.dev/doc/effective_go.html#multiple-returns).)
- Import the Go standard library errors package so you can use its errors.New function.
- Add an if statement to check for an invalid request (an empty string where the name should be) and return an error if the request is invalid. The `errors.New` function returns an error with your message inside.
- Add nil (meaning no error) as a second value in the successful return. That way, the caller can see that the function succeeded.

2. In your hello/hello.go file, handle the error now returned by the `Hello` function, along with the non-error value.

Paste the following code into [hello/hello.go](hello/hello.go).

```go
package main

import (
    "fmt"
    "log"

    "example.com/greetings"
)

func main() {
    // Set properties of the predefined Logger, including
    // the log entry prefix and a flag to disable printing
    // the time, source file, and line number.
    log.SetPrefix("greetings: ")
    log.SetFlags(0)

    // Request a greeting message.
    message, err := greetings.Hello("")
    // If an error was returned, print it to the console and
    // exit the program.
    if err != nil {
        log.Fatal(err)
    }

    // If no error was returned, print the returned message
    // to the console.
    fmt.Println(message)
}
```

In this code, you:

- Configure the `log` package to print the command name `("greetings: ")` at the start of its log messages, without a time stamp or source file information.
- Assign both of the `Hello` return values, including the error, to variables.
- Change the `Hello` argument from `Gladys`’s name to an empty string, so you can try out your error-handling code.
- Look for a non-nil error value. There's no sense continuing in this case.
- Use the functions in the standard library's log package to output error information. If you get an error, you use the log package's `Fatal` function to print the error and stop the program.

3. At the command line in the hello directory, run hello.go to confirm that the code works.

Now that you're passing in an empty name, you'll get an error.

```
$ go run .
greetings: empty name
exit status 1
```

That's common error handling in Go: Return an error as a value so the caller can check for it.

Next, you'll use a Go slice to return a randomly-selected greeting.

## Return a random greeting

In this section, you'll change your code so that instead of returning a single greeting every time, it returns one of several predefined greeting messages. 

To do this, you'll use a Go slice. A slice is like an array, except that its size changes dynamically as you add and remove items. The slice is one of Go's most useful types.

You'll add a small slice to contain three greeting messages, then have your code return one of the messages randomly. For more on slices, see [Go slices](https://blog.golang.org/slices-intro) in the Go blog.

1. In [greetings/greetings.go](greetings/greetings.go), change your code so it looks like the following.

```go
package greetings

import (
    "errors"
    "fmt"
    "math/rand"
    "time"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return name, errors.New("empty name")
    }
    // Create a message using a random format.
    message := fmt.Sprintf(randomFormat(), name)
    return message, nil
}

// init sets initial values for variables used in the function.
func init() {
    rand.Seed(time.Now().UnixNano())
}

// randomFormat returns one of a set of greeting messages. The returned
// message is selected at random.
func randomFormat() string {
    // A slice of message formats.
    formats := []string{
        "Hi, %v. Welcome!",
        "Great to see you, %v!",
        "Hail, %v! Well met!",
    }

    // Return a randomly selected message format by specifying
    // a random index for the slice of formats.
    return formats[rand.Intn(len(formats))]
}
```

In this code, you:
- Add a `randomFormat` function that returns a randomly selected format for a greeting message. Note that randomFormat starts with a lowercase letter, making it accessible only to code in its own package (in other words, it's not exported).
- In `randomFormat`, declare a formats slice with three message formats. When declaring a slice, you omit its size in the brackets, like this: `[]string`. This tells Go that the size of the array underlying the slice can be dynamically changed.
- Use the `math/rand` package to generate a random number for selecting an item from the slice.
- Add an `init` function to seed the rand package with the current time. Go executes init functions automatically at program startup, after global variables have been initialized. For more about init functions, see [Effective Go](https://go.dev/doc/effective_go.html#init).
- In `Hello`, call the `randomFormat` function to get a format for the message you'll return, then use the format and name value together to create the message.
- Return the message (or an error) as you did before.

2. In [hello/hello.go](hello/hello.go), change your code so it looks like the following.

You're just adding Gladys's name (or a different name, if you like) as an argument to the `Hello` function call in [hello.go](hello/hello.go).

```go
package main

import (
    "fmt"
    "log"

    "example.com/greetings"
)

func main() {
    // Set properties of the predefined Logger, including
    // the log entry prefix and a flag to disable printing
    // the time, source file, and line number.
    log.SetPrefix("greetings: ")
    log.SetFlags(0)

    // Request a greeting message.
    message, err := greetings.Hello("Gladys")
    // If an error was returned, print it to the console and
    // exit the program.
    if err != nil {
        log.Fatal(err)
    }

    // If no error was returned, print the returned message
    // to the console.
    fmt.Println(message)
}
```

3. At the command line, in the hello directory, run hello.go to confirm that the code works. Run it multiple times, noticing that the greeting changes.

```
$ go run .
Great to see you, Gladys!
```

```
$ go run .
Hi, Gladys. Welcome!
```

```
$ go run .
Hail, Gladys! Well met!
```

Next, you'll use a slice to greet multiple people. 

## Return greetings for multiple people

In the last changes you'll make to your module's code, you'll add support for getting greetings for multiple people in one request. In other words, you'll handle a multiple-value input, then pair values in that input with a multiple-value output. To do this, you'll need to pass a set of names to a function that can return a greeting for each of them.

But there's a hitch. Changing the Hello function's parameter from a single name to a set of names would change the function's signature. If you had already published the `example.com/greetings` module and users had already written code calling Hello, that change would break their programs.

In this situation, a better choice is to write a new function with a different name. The new function will take multiple parameters. That preserves the old function for backward compatibility. 

1. In [greetings/greetings.go](greetings/greetings.go), change your code so it looks like the following.

```go
package greetings

import (
    "errors"
    "fmt"
    "math/rand"
    "time"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return name, errors.New("empty name")
    }
    // Create a message using a random format.
    message := fmt.Sprintf(randomFormat(), name)
    return message, nil
}

// Hellos returns a map that associates each of the named people
// with a greeting message.
func Hellos(names []string) (map[string]string, error) {
    // A map to associate names with messages.
    messages := make(map[string]string)
    // Loop through the received slice of names, calling
    // the Hello function to get a message for each name.
    for _, name := range names {
        message, err := Hello(name)
        if err != nil {
            return nil, err
        }
        // In the map, associate the retrieved message with
        // the name.
        messages[name] = message
    }
    return messages, nil
}

// Init sets initial values for variables used in the function.
func init() {
    rand.Seed(time.Now().UnixNano())
}

// randomFormat returns one of a set of greeting messages. The returned
// message is selected at random.
func randomFormat() string {
    // A slice of message formats.
    formats := []string{
        "Hi, %v. Welcome!",
        "Great to see you, %v!",
        "Hail, %v! Well met!",
    }

    // Return one of the message formats selected at random.
    return formats[rand.Intn(len(formats))]
}
```

In this code, you:

- Add a `Hellos` function whose parameter is a slice of names rather than a single name. Also, you change one of its return types from a `string` to a `map` so you can return names mapped to greeting messages.
- Have the new `Hellos` function call the existing `Hello` function. This helps reduce duplication while also leaving both functions in place.
- Create a `messages` map to associate each of the received names (as a key) with a generated message (as a value). In Go, you initialize a map with the following syntax: `make(map[key-type]value-type)`. You have the `Hellos` function return this map to the caller. For more about maps, see [Go maps in action](https://blog.golang.org/maps) on the Go blog.
- Loop through the names your function received, checking that each has a non-empty value, then associate a message with each. In this for loop, range returns two values: the index of the current item in the loop and a copy of the item's value. You don't need the index, so you use the Go blank identifier (an underscore) to ignore it. For more, see [The blank identifier](https://go.dev/doc/effective_go.html#blank) in Effective Go.

2. In your [hello/hello.go](hello/hello.go) calling code, pass a slice of names, then print the contents of the names/messages map you get back.

In [hello/hello.go](hello/hello.go), change your code so it looks like the following.

```go
package main

import (
    "fmt"
    "log"

    "example.com/greetings"
)

func main() {
    // Set properties of the predefined Logger, including
    // the log entry prefix and a flag to disable printing
    // the time, source file, and line number.
    log.SetPrefix("greetings: ")
    log.SetFlags(0)

    // A slice of names.
    names := []string{"Gladys", "Samantha", "Darrin"}

    // Request greeting messages for the names.
    messages, err := greetings.Hellos(names)
    if err != nil {
        log.Fatal(err)
    }
    // If no error was returned, print the returned map of
    // messages to the console.
    fmt.Println(messages)
}
```

With these changes, you:

- Create a names variable as a slice type holding three names.
- Pass the names variable as the argument to the Hellos function.

3. At the command line, change to the directory that contains [hello/hello.go](hello/hello.go), then use `go run` to confirm that the code works.

The output should be a string representation of the map associating names with messages, something like the following:

```
$ go run .
map[Darrin:Hail, Darrin! Well met! Gladys:Hi, Gladys. Welcome! Samantha:Hail, Samantha! Well met!]
```

This topic introduced maps for representing name/value pairs. It also introduced the idea of preserving backward compatibility by implementing a new function for new or changed functionality in a module. For more about backward compatibility, see Keeping your modules compatible.

Next, you'll use built-in Go features to create a unit test for your code.

## Add a test

Now that you've gotten your code to a stable place (nicely done, by the way), add a test. Testing your code during development can expose bugs that find their way in as you make changes. In this topic, you add a test for the `Hello` function. 

Go's built-in support for unit testing makes it easier to test as you go. Specifically, using naming conventions, Go's `testing` package, and the `go test` command, you can quickly write and execute tests.

1. In the [greetings](greetings) directory, create a file called [greetings_test.go](greetings/greetings_test.go).

Ending a file's name with `_test.go` tells the go test command that this file contains test functions.

2. In [greetings_test.go](greetings/greetings_test.go), paste the following code and save the file.

```go
package greetings

import (
    "testing"
    "regexp"
)

// TestHelloName calls greetings.Hello with a name, checking
// for a valid return value.
func TestHelloName(t *testing.T) {
    name := "Gladys"
    want := regexp.MustCompile(`\b`+name+`\b`)
    msg, err := Hello("Gladys")
    if !want.MatchString(msg) || err != nil {
        t.Fatalf(`Hello("Gladys") = %q, %v, want match for %#q, nil`, msg, err, want)
    }
}

// TestHelloEmpty calls greetings.Hello with an empty string,
// checking for an error.
func TestHelloEmpty(t *testing.T) {
    msg, err := Hello("")
    if msg != "" || err == nil {
        t.Fatalf(`Hello("") = %q, %v, want "", error`, msg, err)
    }
}
```

In this code, you:

- Implement test functions in the same package as the code you're testing.
- Create two test functions to test the `greetings.Hello` function. Test function names have the form `TestName`, where Name says something about the specific test. Also, test functions take a pointer to the testing package's `testing.T` type as a parameter. You use this parameter's methods for reporting and logging from your test.
- Implement two tests:
  - `TestHelloName` calls the Hello function, passing a name value with which the function should be able to return a valid response message. If the call returns an error or an unexpected response message (one that doesn't include the name you passed in), you use the t parameter's `Fatalf` method to print a message to the console and end execution.
  - `TestHelloEmpty` calls the `Hello` function with an empty string. This test is designed to confirm that your error handling works. If the call returns a non-empty string or no error, you use the t parameter's `Fatalf` method to print a message to the console and end execution.

3. At the command line in the greetings directory, run the `go test` command to execute the test.

The go test command executes test functions (whose names begin with `Test`) in test files (whose names end with `_test.go`). You can add the `-v` flag to get verbose output that lists all of the tests and their results.

The tests should pass.

```
$ go test
PASS
ok      example.com/greetings   0.002s
```

```
$ go test -v
=== RUN   TestHelloName
--- PASS: TestHelloName (0.00s)
=== RUN   TestHelloEmpty
--- PASS: TestHelloEmpty (0.00s)
PASS
ok      example.com/greetings   0.002s

```

4. Break the `greetings.Hello` function to view a failing test.

The `TestHelloName` test function checks the return value for the name you specified as a `Hello` function parameter. To view a failing test result, change the `greetings.Hello` function so that it no longer includes the name.

In [greetings/greetings.go](greetings/greetings.go), paste the following code in place of the `Hello` function. Note that the highlighted lines change the value that the function returns, as if the name argument had been accidentally removed.

```go
// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return name, errors.New("empty name")
    }
    // Create a message using a random format.
    // message := fmt.Sprintf(randomFormat(), name)
    message := fmt.Sprint(randomFormat())
    return message, nil
}
```

5. At the command line in the greetings directory, run go test to execute the test.

This time, run `go test` without the `-v` flag. The output will include results for only the tests that failed, which can be useful when you have a lot of tests. The `TestHelloName` test should fail -- `TestHelloEmpty` still passes.

```
$ go test
--- FAIL: TestHelloName (0.00s)
    greetings_test.go:15: Hello("Gladys") = "Hi, %v. Welcome!", <nil>, want match for `\bGladys\b`, nil
FAIL
exit status 1
FAIL    example.com/greetings   0.002s

```

In the next (and last) topic, you'll see how to compile and install your code to run it locally.

6. Fix the broken `greetings.Hello` function to pass the tests again.

In [greetings/greetings.go](greetings/greetings.go), switch the comment bewteen the declarations of the variable `message`. It must look like here:


```go
    message := fmt.Sprintf(randomFormat(), name)
    // message := fmt.Sprint(randomFormat())
```

Run the tests to ensure they're being passed:

```
$ go test
PASS
ok      example.com/greetings   0.002s

```

```
$ go test -v
=== RUN   TestHelloName
--- PASS: TestHelloName (0.00s)
=== RUN   TestHelloEmpty
--- PASS: TestHelloEmpty (0.00s)
PASS
ok      example.com/greetings   0.002s
```

## Compile and install the application

In this last topic, you'll learn a couple new go commands. While the go run command is a useful shortcut for compiling and running a program when you're making frequent changes, it doesn't generate a binary executable.

This topic introduces two additional commands for building code:

- The [go build](https://go.dev/cmd/go/#hdr-Compile_packages_and_dependencies) command compiles the packages, along with their dependencies, but it doesn't install the results.
- The [go install](https://go.dev/ref/mod#go-install) command compiles and installs the packages.

1. From the command line in the [hello](hello) directory, run the `go build` command to compile the code into an executable.

```
$ go build
```

From the command line in the [hello](hello) directory, run the new [hello](hello/hello) executable to confirm that the code works.

Note that your result might differ depending on whether you changed your [greetings.go](greetings/greetings.go) code after testing it.

On Linux or Mac:

```
$ ./hello
map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
```

On Windows:

```
$ hello.exe
map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
```

You've compiled the application into an executable so you can run it. But to run it currently, your prompt needs either to be in the executable's directory, or to specify the executable's path.

Next, you'll install the executable so you can run it without specifying its path.

3. Discover the Go install path, where the go command will install the current package.

You can discover the install path by running the go list command, as in the following example:

```
$ go list -f '{{.Target}}'
/home/joseba/.gvm/pkgsets/go1.17.5/global/bin/hello
```

For example, the command's output might say `/home/gopher/bin/hello`, meaning that binaries are installed to `/home/gopher/bin`. You'll need this install directory in the next step. 

4. Add the Go install directory to your system's shell path.

    ⚠️  If you're using [GVM](https://github.com/moovweb/gvm) you can omit this step, because GVM automatically adds the current Go version binaries into the `PATH` environment variable.

That way, you'll be able to run your program's executable without specifying where the executable is.

On Linux or Mac, run the following command:

```
$ export PATH=$PATH:/path/to/your/install/directory
```

On Windows, run the following command:
```
$ set PATH=%PATH%;C:\path\to\your\install\directory
```

As an alternative, if you already have a directory like `$HOME/bin` in your shell path and you'd like to install your Go programs there, you can change the install target by setting the `GOBIN` variable using the go env command:

```
$ go env -w GOBIN=/path/to/your/bin
```

or

```
$ go env -w GOBIN=C:\path\to\your\bin
```

5. Once you've updated the shell path, run the go install command to compile and install the package.

```
$ go install
```

6. Run your application by simply typing its name. To make this interesting, open a new command prompt and run the `hello` executable name in some other directory.

```
$ hello
map[Darrin:Hail, Darrin! Well met! Gladys:Great to see you, Gladys! Samantha:Hail, Samantha! Well met!]
```

That wraps up this Go tutorial!

## Conclusion

In this tutorial, you wrote functions that you packaged into two modules: one with logic for sending greetings; the other as a consumer for the first.

For more on managing dependencies in your code, see [Managing dependencies](https://go.dev/doc/modules/managing-dependencies). For more about developing modules for others to use, see [Developing and publishing modules](https://go.dev/doc/modules/developing).

For many more features of the Go language, check out the [Tour of Go](https://go.dev/tour/).
