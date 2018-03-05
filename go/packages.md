[TOC]

## Packages

##### package naming convention

- The convention for naming your package is to use the name of the directory containing it
- Keep in mind that a unique name is not required, because you import the package using its full path.

---

##### package main

- The package name main has special meaning in Go. It designates to the Go command that this package is intended to be compiled into a binary executable.  All of the executable programs you build in Go must have a package called main.

```go
package main
import "fmt"

func main(){
  fmt.Println("Hello World")
}
```

had you named the package something else other than main, let's say hello, then you had been telling the compiler that this is just a package name not a command.

---

##### imports

- If Go was installed under /usr/local/go and your GOPATH was set to /home/myproject:/home/mylibraries, the compiler would look for the net/http package in the following order:      

  ![image](https://www.safaribooksonline.com/library/view/go-in-action/9781617291784/042fig02_alt.jpg)

Go installation directory is the first place the compiler looks and then each directory listed in your GOPATH in the order that they’re listed.  

- `import "github.com/spf13/viper"` the go build command will search the GOPATH for this package location on disk. The fact that it represents a URL to a repository on GitHub is irrelevant as far as the         go build command is concerned. When an import path contains a URL, the *Go tooling can be used to fetch the package from the DVCS and  place the code inside the GOPATH at the location that matches the URL.* This fetching is done using the `go get` command. **`go get` is recursive**
- If you want to import multiple packages with the same name use `named imports`

```go
package main
import (
	"fmt"
	myfmt "mylib/fmt"
    _ "mylib/onlyinitfuncused/fmt"
)
```

- The Go compiler will fail the build and output an error whenever you import a package that you don’t use.



> The _ (underscore character) is known as the blank identifier and has many uses within Go. It’s used when you want to throw away the assignment of a value, including the assignment of  an import to its package name, or ignore return values from a function when you’re only interested in the others.      

---

##### init

- A package may contain many `func init()`. All the init functions that are discovered by the compiler are scheduled to be executed prior to the main function.  
- The init functions are great for setting up packages, initializing variables, or performing any other bootstrapping you may need prior  to the program running.      

An example of this is database drivers. They register themselves with the sql package when their init function is executed at startup because the sql package can’t know about the drivers that exist when it’s compiled. Let’s look at an example of what an init function might do.

​      

![image](https://www.safaribooksonline.com/library/view/go-in-action/9781617291784/044fig01_alt.jpg)



 As stated earlier, you can’t import a package that you aren’t using, so renaming the import with the blank identifier allows the init function to be discovered and scheduled to run without the compiler issuing an error about unused imports. 

![image](https://www.safaribooksonline.com/library/view/go-in-action/9781617291784/045fig01_alt.jpg)

---

##### go tools

###### go build

- compiles packages and dependencies and builds binary.
- `go build`  You can omit the filename of the source code file that you want to build, and the go tool will default to the current package.
- `go build github.com/goinaction/code/chapter3/wordcount`  packages can be specified directly.
- `go build github.com/goinaction/code/chapter3/...` wildcards allowed

###### go run

The go run command both builds and executes the program contained in wordcount.go, which saves a lot on typing.      

###### go clean

removes the executable created.

###### go vet

it catches error

###### go format

for formatting the code

###### go get

- go get installs package at $GOPATH/src directory so we will have to use the same version of a package for different projects as all our projects are under a single directory. So each of the projects will not have different versions of dependencies with them.
- We won't be able to determine which version of package we need unless it is hosted in complete different repository as go get always fetches from the latest version of a package. This leads to problem when working as team we might end up fetching different version of a package.

---

##### package management 

The most famous tools today are **godep, vendor and glide**

