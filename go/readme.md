# DIT Go Coding standard

## Code style and format
* Avoid global variables, even in packages. By doing so you introduce side effects if the package is included multiple times.

* Use [goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports) before committing. goimports is a tool that automatically formats Go source code using [Gofmt](https://golang.org/cmd/gofmt/), in addition to formatting import lines, adding missing ones and removing unreferenced ones.

* Most editors/IDEs allow you to run commands before/after saving a file, you can set it up to run [goimports]() so that it’s applied to every file when saving.

* Place private methods below the first caller method in the source file

### Naming
#### Package Names
When naming packages, choose a name that is:
* All lower-case. No capitals or underscores.
* Does not need to be renamed using named imports at most call sites.
* Short and succinct. Remember that the name is identified in full at every call site.
* Not plural. For example, net/url, not net/urls.
* Not "common", "util", "shared", or "lib". These are bad, uninformative names.

See also [Package Names](https://blog.golang.org/package-names) for more info.

#### Function Names
We follow the Go community's convention of using MixedCaps for function names. An exception is made for test functions, which may contain underscores for the purpose of grouping related test cases, e.g TestMyFunction_WhatIsBeingTested.

### Model, View and Controller Names
Model struct names should be singular and should be stored in models folder and should be in models module. For example
```go
//file name: role.go
package models
type Role struct {
	Model              // common to all tables
	Name        string `json:"name"`
	Description string `json:"description"`
}
```

Views should be singular as well and should be stored in /views folders. For example, index view for role should be stored in a foloder /views/role/index.html

```go
//file name: index.go
package views
//...
//...
```

Controllers should be stored in controllers folder and should have a singular name. They should be in controllers module.
```go
//file name: role.go
package controllers
import "github.com/labstack/echo/v4"

type Role{
	//...
}

//Index function
func (r *Role) Index (c echo.Context) error{
	//...
}

//create function
func (r *Role) Create (c echo.Context) error{
	//...
}

```

### File names
All file names should be small letter and separated by underscore in case the file name contains more than two words. For example, examination_result.go


## Code Testing and Benchmarking
Unit testing is an important part of writing principled Go programs. When writing programme, let us try as much as possible to write unit testing for each function.

Testing should be in a separate file. For example is if you have your functions in result.go, all respective testing should be in result_test.go

### Testing
1. We should not use any specific library or framework for testing, as the [testing standard library](https://golang.org/pkg/testing/) provides already everything needed for testing. If there is a need for more sophisticated testing tools, use [Testify](https://github.com/stretchr/testify) framework.

2. Using [Table-Driven Tests](https://github.com/golang/go/wiki/TableDrivenTests) is generally good practice when you have multiple entries of inputs/outputs for the same function. Below are some guidelines one can follow when writing table-driven test. These guidelines are mostly extracted from Go standard library source code. Keep in mind it’s OK not to follow these guidelines when it makes sense.

3. Contents of the test case
    * Ideally, each test case should have a field with a unique identifier to use for naming subtests. In the Go standard library, this is commonly the name string field.
    * Use want/expect/actual when you are specifying something in the test case that is used for assertion. 

4. Variable names
    * Each table-driven test map/slice of struct can be named tests.
    * When looping through tests the anonymous struct can be referred to as tt or tc.
    * The description of the test can be referred to as name/testName/tn

### Bechmarking

Programs handling a lot of IO or complex operations should always include benchmarks, to ensure performance consistency over time. Use the [testing standard library](https://golang.org/pkg/testing/)


### Testing and Benchmarking example

Typical example of a  function and its respective testing and bechmarking is given. It is slightly modified from [Go by Examples](https://gobyexample.com/testing):

```go
//filename: result.go
package result

func intMin(a, b int) int {
	if a < b {
		return a
	}
	return b
}

```

```go
//filename: result_test.go
package result

import (
	"fmt"
	"testing"
)

//TestIntMinBasic function test intMin for one case
func TestIntMinBasic(t *testing.T) {
	ans := intMin(2, -2)
	if ans != -2 {

		t.Errorf("IntMin(2, -2) = %d; want -2", ans)
	}
}

//TestIntMinTableDriven tests intMin function with different test cases
func TestIntMinTableDriven(t *testing.T) {
	var tests = []struct {
		a, b int
		want int
	}{
		{0, 1, 0},
		{1, 0, 0},
		{2, -2, -2},
		{0, -1, -1},
		{-1, 0, -1},
	}

	for _, tt := range tests {

		testname := fmt.Sprintf("%d,%d", tt.a, tt.b)
		t.Run(testname, func(t *testing.T) {
			ans := intMin(tt.a, tt.b)
			if ans != tt.want {
				t.Errorf("got %d, want %d", ans, tt.want)
			}
		})
	}
}

//BechcmarkIntMin bencharms the int function
func BenchmarkIntMin(b *testing.B) {
	for i := 0; i < b.N; i++ {
		intMin(i, i)
	}
}

```

## Error handling

### Adding context
Adding context before you return the error can be helpful, instead of just returning the error. This allows developers to understand what the program was trying to do when it entered the error state making it much easier to debug.

For example: 
```go
// Wrap the error
return nil, fmt.Errorf("get cache %s: %w", f.Name, err)

// Just add context
return nil, fmt.Errorf("saving cache %s: %v", f.Name, err)

```

A few things to keep in mind when adding context:
* Decide if you want to expose the underlying error to the caller. If so, use %w, if not, you can use %v.
* Don’t use words like failed, error, didn't. As it’s an error, the user already knows that something failed and this might lead to having strings like failed xx failed xx failed xx. Explain what failed instead.
* Error strings should not be capitalized or end with punctuation or a newline.


### Checking Error types

* To check error equality don’t use ==. Use errors.Is instead (for Go versions >= 1.13).
* To check if the error is of a certain type don’t use type assertion, use errors.As instead (for Go versions >= 1.13). 

### References for working with errors

* [Programing with errors.](https://peter.bourgon.org/blog/2019/09/11/programming-with-errors.html)
* [Don’t just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)


## Recommended Libraries
The following are recommneded libraries



## Code Documentation

The source should be properly documented by following go simple documentation conversion standard: to document a type, variable, constant, function, or even a package, write a regular comment directly preceding its declaration, with no intervening blank line. [Godoc](https://golang.org/cmd/godoc/) will then present that comment as text alongside the item it documents. For example, this is the documentation for the fmt package's Fprint function:
```go
// Fprint formats using the default formats for its operands and writes to w.
// Spaces are added between operands when neither is a string.
// It returns the number of bytes written and any write error encountered.
func Fprint(w io.Writer, a ...interface{}) (n int, err error) {
	//...
}

```
Documentation generation should be made using godoc tool
