#### arrays

- The type of an array variable includes both the length and the type of data that can be stored in each element. Only arrays of the same type can be assigned.

```go
// When an array is initialized, each individual element that belongs to the array is initialized to its zero value
var array [5]int

// an array literal
array := [5]int{1, 2, 3, 4, 5}

// an array where go calulate sizes
array := [...]int{1, 2, 3, 4}

// initializing only specific elements, here 0th, 2nd, 3rd are initialized to zero
array:= [5]int{1: 10, 2: 10} 

// 
```

- Lets see array of strings, we know a string is a `two word datastructure`, one bute is pointer to a backing byte array, while other byte store the length. Since the string is of constant length it can find itself on stack frames.

```go
// string  --> | pointer to backing byte array |
//			   | len 						   |
// array zero value   --->  |nil|nil|nil|nil|nil|
							| 0 | 0 | 0 | 0 | 0 |		
var array [5]string 

// everything in go is pass by value
array[0] = "Apple"
```

![image](./pkg/dsa-array-1.jpg)



