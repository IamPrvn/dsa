[TOC]



#### use of keyword `var` for zero values

Use keyword `var` as a way to indicate that variable is initialized to its zero value. If the variable is initialized to something other than its zero value, then use the short variable declaration operator with  a struct literal.

```go
// this will mark the first word of slice to a nil pointer
var slice []int
// the pointer will hold a addrees which will have the type's zero value.
slice := []int{}**



```

---



#### ways to declare userdefined types 

##### using struct

```go
// a user-defined type
type user struct {
  name string
  email string
  ext int
  priviledged bool
}

// declaration of variable of struct type set to its zero value
var bill user

// Declaration of a variable of the struct type using a struct literal
// observe the trailing comma , the variables can be written in any order
lisa := user{
  name: "Lisa",
  email: "lisa@email.com",
  ext: 123,
  priviledged: true,
}

// another way to declare a stuct literal
lisa := user{"Lisa", "lisa@email.com", 123, true}
```

##### using as type specification for the new type

```go
type Duration int64

//note duration and int64 are different types even though int64 is the base type of Duration
var dur Duration
dur = int64(1000)  //error
```

###### Why do we need to create a user defined type from a primitive type?

Some times we need to create functions around primitive types define behaviour, compiler will only let you declare methods for user-defined types that are named.

 

---



#### methods

When a function has a receiver it is called method. Receiver are of two types:-

1. value receiver
2. pointer receiver