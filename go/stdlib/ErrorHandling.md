[TOC]



# Error handling in golang



## Sentinel errors values



A sentinel is used to describe an event using some value, that no further processing is possible. So to with Go, we use specific values to signify an error.



### Problem



#### Using  `if err == ErrSomething { … }`

- As and when caller compares the err to predifined error type using eqaulity operator. This presents a problem when you want to provide more context, as returning a different error would will break the equality check.
- So, instead the caller will be forced to look at the output of the `error.Error()` to see if it matches a specific string.

#### Never inspect the output of `error.Error`

- Comparing the string form of an error is, in my opinion, a code smell, and you should try to avoid it. The Error method on the error interface exists for humans, not code. The contents of that string belong in a log file, or displayed on screen. You shouldn’t try to change the behaviour of your program by inspecting it.

#### Sentinel errors create a dependency between two packages

- imagine the coupling that exists when many packages in your project export error values, which other packages in your project must import to check for specific error conditions.

#### Sentinel errors becoe part of your public api

- If your API defines an interface which returns a specific error, all implementations of that interface will be restricted to returning only that error, even if they could provide a more descriptive error.

  We see this with io.Reader. Functions like io.Copy require a reader implementation to return exactly io.EOF to signal to the caller no more data, but that isn’t an error.



### Solutions

- Avoid using sentinel error values.



## Sentinel error types

- Error types are another way declaring error in go which gives more context..

  ```go
  type MyError struct {
          Msg string
          File string
          Line int
  }

  func (e *MyError) Error() string { 
          return fmt.Sprintf("%s:%d: %s”, e.File, e.Line, e.Msg)
  }

  return &MyError{"Something happened", “server.go", 42}
                  
                  
  // you can then use type assertion..
  err := something()
  switch err := err.(type) {
  case nil:
          // call succeeded, nothing to do
  case *MyError:
          fmt.Println(“error occurred on line:”, err.Line)
  default:
  // unknown error
  }
  ```



### Problems

- same as [Sentinel errors create a dependency between two packages](#sentinel-errors-create-a-dependency-between-two-packages)

  ​



### 

