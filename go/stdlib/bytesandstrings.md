[TOC]



In [iopackages](./iopackages.md), we mostly talked about byte streams, here we will talk about bounded, in memory byte slices instead.

### A brief aside on bytes vs strings

Rob Pike has an excellent, thorough post on [strings, bytes, runes, and characters](https://blog.golang.org/strings) but for the sake of this post I’d like to provide more concise definitions from an application developer standpoint.

- Byte slices represent a mutable, resizable, contiguous list of bytes. That’s a mouthful so let’s understand what that means.

Given a slice of bytes:

```
buf := []byte{1,2,3,4}
```

It’s mutable so you can update elements:

```
buf[3] = 5  // []byte{1,2,3,5}
```

It’s resizable so you can shrink it or grow it:

```
buf = buf[:2]           // []byte{1,2}
buf = append(buf, 100)  // []byte{1,2,100}
```

> Always try to create byte slices which have their length == capacity, in this way if we will append a byte slice it will create a new byte slice rather than changing the backing byte slice.

And it’s contiguous so each byte exists one after another in memory:

```
1|2|3|4
```

- Strings, on the other hand, represent an *immutable*, *fixed-size*, contiguous list of bytes. That means that you can’t update a string — you can only create new ones. This is important from a performance standpoint. In high performance code, constantly creating new strings adds a lot of load on the garbage collector.
- From an application development perspective,
  -  strings tend to be easier to use when working with UTF-8 data, they can be used as map keys whereas byte slices cannot.
  -  most APIs use strings for arguments containing character data. 
  -  On the other hand, byte slices work well when you’re dealing with raw bytes such as processing byte streams. 
  -  They are also good to use when you need to avoid allocations and can reuse them.

---

#### Adapting strings and slices for streams

bytes and strings packages  provide a way to interface in-memory byte slices and strings as io.[Reader](https://golang.org/pkg/io/#Reader) and io.[Writers](https://golang.org/pkg/io/#Writer).

##### In-memory readers

- Two of the most underused tools in the Go standard library are the bytes.[NewReader](https://golang.org/pkg/bytes/#NewReader) and strings.[NewReader](https://golang.org/pkg/strings/#NewReader) functions:

```
func NewReader(b []byte) *Reader
func NewReader(s string) *Reader
```

- These functions return an io.[Reader](https://golang.org/pkg/io/#Reader) implementation that wraps around your in-memory byte slice or string. But these aren’t just readers — they implement all the read-related interfaces in [io](https://golang.org/pkg/io/) including io.[ReaderAt](https://golang.org/pkg/io/#ReaderAt), io.[WriterTo](https://golang.org/pkg/io/#WriterTo), io.[ByteReader](https://golang.org/pkg/io/#ByteReader), io.[ByteScanner](https://golang.org/pkg/io/#ByteScanner), io.[RuneReader](https://golang.org/pkg/io/#RuneReader), io.[RuneScanner](https://golang.org/pkg/io/#RuneScanner), & io.[Seeker](https://golang.org/pkg/io/#Seeker).
- I frequently see code where byte slices or strings are written to a bytes.[Buffer](https://golang.org/pkg/bytes/#Buffer) and then the buffer is used as a reader:

```go
var buf bytes.Buffer
buf.WriteString("foo")
http.Post("http://example.com/", "text/plain", &buf)
```

​      However, this approach incurs heap allocations which will be slow and use additional memory. A    better option is to use the strings.[Reader](https://golang.org/pkg/strings/#Reader):

```go
r := strings.NewReader("foobar")
http.Post("http://example.com", "text/plain", r)
```

​    This approach also works when you have multiple strings or byte slices by using the io.MultiReader:

```go
r := io.MultiReader(
        strings.NewReader("HEADER"),
        bytes.NewReader([]byte{0,1,2,3,4}),
        myFile,
        strings.NewReader("FOOTER"),
)
```

##### In-memory writers

- For this we extensively use bytes package.
- A [bytes.Buffer](http://golang.org/pkg/bytes/#Buffer) satisfies both the Writer and Reader interfaces, which makes sense, because a buffer can be both written to and read from.

##### caveats with bytes.Buffer

- This works just fine in most cases, when you have small enough data that the memory overhead isn’t that onerous, a couple hundred bytes or so, up to a few KB.
- That bytes.Buffer starts off small (64 bytes), and grows as needed as data is written to it, possibly allocating a whole new slice and copying your data into it, then discarding the original byte slice to be garbage collected.
- You could specify the size of the buffer with [bytes.NewBuffer](http://golang.org/pkg/bytes/#NewBuffer), but even then you won’t know exactly how many bytes you’ll end up writing, and you may overallocate or underallocate.

---

#### strings in go

1. strings are made of up runes (not bytes)

   so instead of ranging over bytes, we should range over strings, so that you do not lose emoji's and special characters. [refer this](https://mymemorysucks.wordpress.com/2017/05/03/a-short-guide-to-mastering-strings-in-golang/)

2. characters/runes in a string are of variable length (rune is basically a spaceholder of 4 bytes, an alias of int32)

3. ranging over a string with a for loop returns runes

4. strings are immutable while a []rune slice is mutable


---



#### Package Organization

Both string and bytes packages appear to be large, but we will look into them by categorizing :

- Comparison functions
- Inspection functions
- Prefix/suffix functions
- Replacement functions
- Splitting and joining functions



##### Comparison Functions

###### Equality

```go
func Equal(a, b []byte) bool
```

- Equal function only exist in the byte package , strings are compared for equality using `==`.

- Prefer `EqualFolds` over this will create 2 allocations of new strings 

  ```go
  if strings.ToUpper(a) == strings.ToUpper(b) {
         return true
  }
  ```


###### Comparison

```go
func compare(a, b []byte) int
func compare(a, b string) int
```

- -1,0, 1 are the return values.

- We should not use compare function for strings, its only their for completeness.

- `sort.Interface` requires compare method for its `Less()` function.

  ``` go
  type ByteSlices []byte

  func (b ByteSlices) Less(i, j) bool{
    return bytes.Compare(b[i], b[j]) == -1
  }
  ```

###### Inspection

- If a byte/string contains a subslice :-

  ```go
  func Contains(b, sublice []byte) bool
  func Contains(s, substring stirng) bool
  ```

- Number of occurences of a subslice/substring in a byte/string

  ```go
  func Count(s, sep []byte) int
  func Count(s, sep string) int
  ```

- Another use for [Count](https://golang.org/pkg/bytes/#Count)() is to return the number of runes in a string. By passing in an empty slice or blank string as the *sep* argument, [Count](https://golang.org/pkg/bytes/#Count)() will return the number of runes + 1.

- indexing functions :-

  ```go
  Index(s, sep []byte) int
  IndexAny(s []byte, chars string) int
  IndexByte(s []byte, c byte) int
  IndexFunc(s []byte, f func(r rune) bool) int
  IndexRune(s []byte, r rune) int
  ```

  Similarly for above all there is a LastIndex() variant too.

- Checking for prefixes/suffixes

  ```go
  func HasPrefix(s, prefix []byte) bool
  func HasPrefix(s, prefix string) bool
  func HasSuffix(s, suffix []byte) bool
  func HasSuffix(s, suffix string) bool
  ```

- Trimming fucntions in go

  ```go
  func trim(s []byte, cutset []byte) []byte
  func trim(s, cutset string) string
  ```

  This will remove any runes in *cutset* from the beginning and end of your string. You can also trim from just the beginning or just the end of your string using [TrimLeft](https://golang.org/pkg/bytes/#TrimLeft)() and [TrimRight](https://golang.org/pkg/bytes/#TrimRight)(), respectively.

  But generic trimming isn’t very common. Most of the time you want to trim white space characters and you can use [TrimSpace](https://golang.org/pkg/bytes/#TrimSpace)() for this:

  ```go
  // this will trim all unicode defined white space which includes space, new line, tab characters, thin space and hair space 
  func TrimSpace(s []byte) []byte
  func TrimSpace(s string) string

  //then there are TrimPerfix and TrimSuffix functions too
  ```

###### Replacement functions

**Basic Replacement**

```go
// n here is number of times the replacement is to be done, -1 all the times
func Replace(s, old, new []byte, n int) []byte
func Replace(s, old, new string, n int) string

now := time.Now().Format(time.Kitchen)
println(strings.Replace(data, "$NOW", now, -1)
        
//If you have multiple mappings then you’ll need to use strings.Replacer. This works by specifying old/new pairs to strings.NewReplacer():
r := strings.NewReplacer("$NOW", now, "$USER", "mary")
println(r.Replace("Hello $USER, it is $NOW"))
```

**Case Replacement**

- Since go works with unicode converting cases is not as trivial, there are 3 types of casing `upper lower and title case`

- uppercase and lower are straight forword for most of the languages but for some languages have different rules for casing, Turkish, for example, uppercases its **i** as **İ**. For these special case languages, there are special versions of these functions.

  ```go
  // below functions are similar for strings
  func ToUpper(b [] bytes) []bytes
  func ToLower(b []bytes) []bytes

  strings.ToUpperSpecial(unicode.TurkishCase, "i")
  ```

- `title case ` in unicode world is different. It is not a way to capatilize the first character in each word.For the most part, title case and upper case are the same but there are a

   few code points which have differences. For example, the [ǉ](https://www.compart.com/en/unicode/U+01C9) code point (yes, that’s one code point) is uppercased as Ǉ but title cased as ǈ.

  ```go
  println(strings.ToTitle("the count of monte cristo"))
  // Output: THE COUNT OF MONTE CRISTO

  //use Title function instead
  println(strings.Title("the count of monte cristo"))
  // Output: The Count Of Monte Cristo
  ```

**Mapping runes**

One other function for replacing data in a bytes slice or string is [Map](https://golang.org/pkg/bytes/#Map)():

```
func Map(mapping func(r rune) rune, s []byte) []byte
func Map(mapping func(r rune) rune, s string) string
```

This function lets you pass in a function to evaluate every rune and replace it.



###### Splitting and Joining functions

**Substring splitting**

```
func Split(s, sep []byte) [][]byte
func SplitAfter(s, sep []byte) [][]byte
func SplitAfterN(s, sep []byte, n int) [][]byte
func SplitN(s, sep []byte, n int) [][]byte
```

```
func Split(s, sep string) []string
func SplitAfter(s, sep string) []string
func SplitAfterN(s, sep string, n int) []string
func SplitN(s, sep string, n int) []string
```

- These break up byte slices or strings by a delimiter and return the subslices or substrings. The “After” functions include the delimiter at the end of the substrings. The “N” functions limit the number of splits that can occur:

```
strings.Split("a:b:c", ":")       // ["a", "b", "c"]
strings.SplitAfter("a:b:c", ":")  // ["a:", "b:", "c"]
strings.SplitN("a:b:c", ":", 2)   // ["a", "b:c"]
```

**Categorical Splitting**

The best example of this is breaking apart words by variable-length whitespace. Simply calling [Split](https://golang.org/pkg/bytes/#Split)()
 using a space delimiter will give you empty substrings if you have multiple contiguous spaces. Instead you can use the Fields() function:

```go
//Fields splits the string s around each instance of one or more consecutive white space
// characters, as defined by unicode.IsSpace, returning a slice of substrings of s or an
// empty slice if s contains only white space.
func Fields(s []byte) [][]byte
```

```go
strings.Fields("hello   world")      // ["hello", "world"]
strings.Split("hello   world", " ")  // ["hello", "", "", "world"]
```

**Joining**

```go
func Join(s [][]byte, sep []byte) []byte
func Join(a []string, sep string) string
```

> Do not try to implement your own join method, mostly this type of flawed code is seen. 
>
> ```
> var output string
> for i, s := range a {
>         output += s
>         if i < len(a) - 1 {
>                 output += ","
>         }
> }
> return output
> ```

The problem with above code is that you are creating a massive number of allocations. because stirngs are immutable each iteration is giving a new string. (`3`)

The `strings.Join()` instead uses buffer something like this :-

```go
func Join(a []string, sep string) string {
	switch len(a) {
	case 0:
		return ""
	case 1:
		return a[0]
	case 2:
		// Special case for common small values.
		// Remove if golang.org/issue/6714 is fixed
		return a[0] + sep + a[1]
	case 3:
		// Special case for common small values.
		// Remove if golang.org/issue/6714 is fixed
		return a[0] + sep + a[1] + sep + a[2]
	}
	n := len(sep) * (len(a) - 1)
	for i := 0; i < len(a); i++ {
		n += len(a[i])
	}

	b := make([]byte, n)
	bp := copy(b, a[0])
	for _, s := range a[1:] {
		bp += copy(b[bp:], sep)
		bp += copy(b[bp:], s)
	}
	return string(b)
}
```

here if number of elements to join are more than 3 then we are creating a byte array first and using copy function.

#### Miscellaneous functions

```go
// repeat functions can be used as a seperator in some log files/terminal
println(strings.Repeat("-", 80))
```

