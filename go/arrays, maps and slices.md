[TOC]



#### arrays

In go all arrays have a known size at compile time, and so all the arrays can find itselves on stack. Hardware loves arrays because of contiguous memory location.

```go
// declaration, gets zero values of int type
var arr [5]int
var array [4][2]int

// array literal
arr := [3]int{1, 3, 5}
array := [4][2]int{{10, 11}, {20, 21}, {30, 31}, {40, 41}}

// declaring array with go calculating sizes
arr := [...]int{1, 3, 6, 8}

// intializing specific elements, here 0 is intialized to int zero value.
arr := [5]int{1: 10, 3: 15}
array := [4][2]int{1: {20, 21}, 3: {40, 41}}

// array pointers
arr := [5]*int{0: new(int), 1: new(int)}
*arr[0] = 10
*arr[1] = 20
```

- The type of an array variable includes both the length and the type of data that can be stored in each element. So  `var a1 [5]int` is differnet from `var a2 [5]int` and `error :- a1 = a2` . Only arrays  of the same type can be assigned. Similarly a string in go is a `two word datastructures, the first word is a pointer to a backing byte array while the second word is the length`. Since both arrays and string length is constant, there value(storage cost) is known at compile time and they can livw on stack while the backing array of bytes can reside on heap.

> slices maps channels interfaces are all reference types, they are  created using make(). all refernce types have nil as there zero value. String zero value is empty not nil. 

---

#### slices

![image](https://www.safaribooksonline.com/library/view/go-in-action/9781617291784/04fig09_alt.jpg)

slices are `3 word datastructures  a pointer to a backing array, length and capacity`

**if you specify a value inside the [ ] operator, you’re creating an array. If you don’t specify a value, you’re creating a slice**

```go
// length and capacity of 5
slice := make([]int, 5)

// length and capacity are different
slice := make([]int, 5 ,7)

// slice literal , initial length and capacity will be #elements
slice := []int{1, 2, 3, 4, 5}
// have an outer slice of two elements that contain an inner slice of integers
slice := [][]int{{10}, {100, 200}}            

// slice literal trick to set initial length and capacity, for a length and capacity of 100 set the 100th index
slice := []string{99: ""}
```

##### nil slice

 as mentioned the zero value of reference type in go is nil, slice is a reference type. They’re useful when you want to represent a slice that doesn’t exist, such as when an exception occurs in a function that returns a slice. A nil slice has a length and capacity of 0 and has no underlying array.

```go
 var slice int[]
```

![image](https://www.safaribooksonline.com/library/view/go-in-action/9781617291784/04fig10_alt.jpg)

##### empty slice

useful when you want to represent an empty collection, such as when a database query returns zero results

```go
slice := make([]int, 0)
slice := []int{}
```

![image](https://www.safaribooksonline.com/library/view/go-in-action/9781617291784/04fig11_alt.jpg)

##### taking slice of slice

```go
// length 5 capacity 5
slice := []int{10, 20 ,30 , 40 ,50}
// length 2 capacity 4
newSlice := slice[1:3]
```

![image](https://www.safaribooksonline.com/library/view/go-in-action/9781617291784/04fig12_alt.jpg)

note view of `newSlice` is different. `newSlice` can’t access the elements of the underlying array that are prior to its pointer. As far as newSlice is concerned, those elements don’t even exist. 

- You need to remember that you now have two slices sharing the same underlying array. Changes made to the shared section of  the underlying array by one slice can be seen by the other slice.

###### growing slices (using append)

- having capacity is great, but you can only access elements upto a slice's length, e.g. in example above `newSlice[4]` will flag an error.

- When your append call returns, it provides you a new slice with the changes. The **append function will always increase the length of the new slice** The capacity, on the other hand, may or may not be affected, depending on the available capacity of the source slice.

  e.g :- `newSlice = append(newSlice, 60)`

  ![image](https://www.safaribooksonline.com/library/view/go-in-action/9781617291784/04fig14_alt.jpg)

- Beacuse there was available capacity, `append func` accommodatd new value in the backing array, but note it's changing the value of the backing array. A safer option would be to create a new backing array whenever append is called. *When there’s no available capacity in the underlying array for a slice, the append function will create a new underlying array, copy the existing values that are being referenced, and assign the new value.*  So when `length = capacity` append will create a new backing array.

  ```go
  source := []string{"Apple", "Banana", "Plum", "Orange", "Grape"}
  slice := source[2:3:3]
  slice := append(slice, "Kiwi")
  ```

  ![image](https://www.safaribooksonline.com/library/view/go-in-action/9781617291784/04fig18_alt.jpg)

When we call append for the first time on the slice, it will create a new underlying array of two elements, copy the fruit Plum, add the new fruit Kiwi, and return a new slice that references this underlying array.

- Multiple values can be appended into a slice at a time,

  ```go 
  // Append the two slices together and display the results. s1 and s2 are slices
  fmt.Printf("%v\n", append(s1, s2...))
  ```

##### passing slice between functions

```go
// Allocate a slice of 1 million integers.
slice := make([]int, 1e6)
// Pass the slice to the function foo.
slice = foo(slice)
// Function foo accepts a slice of integers and returns the slice back.
func foo(slice []int) []int {
  ...    
  return slice
}
```

Passing the 24 bytes between functions is fast and easy. This is the beauty of slices. You don’t need to pass pointers around and deal with complicated syntax. for arrays you would have to pass `foo(&array)`.

---

#### map





- The built-in function **len** can be used to retrieve the length of a slice or map. 
- The built-in function **cap** only works on slices.                                      
- Through the use of composition, you can create multidimensional arrays and slices. You can also create maps with values that are slices and other maps. **A slice can’t be used as a map key.**                                      
- Passing a slice or map to a function is cheap and doesn’t make a copy of the underlying data structure.                     