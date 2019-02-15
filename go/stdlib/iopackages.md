---
title: io packages
---

[TOC]

### Byte Streams

When working with bytes we can do two things , reading and writing.

#### Reading bytes

##### Reader Interface

```go
type Reader interface {
  Read(p []byte) (n int, err error)
}
```

- The interface is implemented throughout the standard library by everything from [network connections](https://golang.org/pkg/net/#Conn) to [files](https://golang.org/pkg/os#File) to [wrappers for in-memory slices](https://golang.org/pkg/bytes/#Reader).
- Buffer p / byte slice p  is passed as argument and not retutned. If Read() returned a byte slice instead of accepting one as an argument then the reader would have to allocate a new byte slice on every Read() call. That would wreak havoc on the garbage collector.
- your buffer isn’t guaranteed to be filled. If you pass an 8-byte slice you could receive anywhere between 0 and 8 bytes back.
- It returns an io.EOF error as a normal part of usage when the stream is done. It may return the (non-nil) error from the same call or return the error (and n == 0) from a subsequent call. An instance of this general case is that a Reader returning a non-zero number of bytes at the end of the input stream may return either err == EOF or err == nil. The next Read should return 0, EOF.
- Callers should always process the n > 0 bytes returned before considering the error err. Doing so correctly handles I/O errors that happen after reading some bytes and also both of the allowed EOF
  behaviors.

---



##### Imporving Reading guarantees

- `Read(buf []byte) (n int,err error)`  returns number of bytes read `n` which is `0 <= n < len(buf)` , these partial reads can be messy, hence some helper functions are required to guarnatee complete reads.

- ```go
  func ReadFull(r Reader, buf []byte) (n int, err error)
  ```

  This function ensures that your buffer is completely filled with data before returning. If your buffer is partially read then you’ll receive an io.ErrUnexpectedEOF back. If no bytes are read then an io.EOF is returned. This simple guarantee simplifies your code tremendously. To read 8 bytes you only need to do this:

  ```go
  buf := make([]byte, 8)
  if _, err := io.ReadFull(r, buf); err == io.EOF {
          return io.ErrUnexpectedEOF
  } else if err != nil {
          return err
  } 
  ```

---



##### Concatenating streams

```go
func MultiReader(readers ...Reader) Reader
```

For example, you may be sending a HTTP request body that combines an in-memory header with data that’s on-disk. Many people will try to copy the header and file into an in-memory buffer but that’s slow and can use a lot of memory.

Here’s a simpler approach:

```go
r := io.MultiReader(
        bytes.NewReader([]byte("...my header...")),
        myFile,
)
http.Post("http://example.com", "application/octet-stream", r)
```

---



##### Duplicating streams

- One issue you may run across when using readers is that once a reader is read, the data cannot be reread. For example, your application may fail to parse an HTTP request body and you’re unable to debug the issue because the parser has already consumed the data.


- ```go
  func TeeReader(r Reader, w Writer) Reader
  ```

  TeeReader returns a Reader that writes to w what it reads from r. All reads from r performed through it are matched with corresponding writes to w. There is no internal buffering - the write must complete before the read completes. Any error encountered while writing is reported as a read error.

- ```go
  var buf bytes.Buffer
  body := io.TeeReader(req.Body, &buf)

  // ... process body ...

  if err != nil {
          // inspect buf
          return err
  }
  ```

---



#### Writing bytes

##### Writer Interface

- The Writer interface is simply the inverse of the Reader. We provide a buffer of bytes to push out onto a stream.

```go
type Writer interface {
        Write(p []byte) (n int, err error)
}
```

- Writing bytes is simpler than reading them. Readers complicate data handling because they allow partial reads, however, partial writes will always return an error

---



##### Duplicating Writes

```go
//MultiWriter creates a writer that duplicates its writes to all the
//    provided writers, similar to the Unix tee(1) command.

func MultiWriter(writers ...Writer) Writer
```

- The name is a bit confusing since it’s not the writer version of MultiReader. Whereas MultiReader concatenates several readers into one, the MultiWriter returns a writer that duplicates each write to multiple writers.

```go
package main
import (
	"fmt"
	"os"
	"io"
)
func main() {
	file1, _ := os.Create("file1.txt")
	file2, _ := os.Create("file2.txt")
	writer :=  io.MultiWriter(file1, file2, os.Stdout)
	buf := []byte("hello there")
	if _, err := writer.Write([]byte(buf)); err != nil {
		fmt.Println(err)
	}
}
```



##### Optimizing Writes when directly writing strings

Sometimes we directly need to write strings to writers. We can do something like this `w.write([]byte("sample_string"))`	, which works as expected. However this is a very frequent operation and so there is a "generally" accepted method for this captured by the `io.stringWriter` unexported interface:

```
type stringWriter interface {
    WriteString(s string) (n int, err error)
}
```

This method gives the possibility to write a `string` value instead of a `[]byte`. So if something (that also implements `io.Writer`) implements this method, you can simply pass `string` values without `[]byte` conversion. *This seems like a minor simplification in code, but it's more than that.* Converting a `string` to `[]byte` has to make a copy of the `string` content (because `string` values are immutable in Go), so there is some overhead which becomes noticeable if the `string` is "bigger" and/or you have to do this many times.

Depending on the nature and implementation details of an `io.Writer`, it may be that it is possible to write the content of a `string` without converting it to `[]byte` and thus avoiding the above mentioned overhead.

As an example, if an `io.Writer` is something that writes to an in-memory buffer (`bytes.Buffer` is such an example), it may utilize the builtin [`copy()`](https://golang.org/pkg/builtin/#copy) function:

> The copy built-in function copies elements from a source slice into a destination slice. **(As a special case, it also will copy bytes from a string to a slice of bytes.)**

The `copy()` may be used to copy the content (bytes) of a `string` into a `[]byte` without converting the `string` to `[]byte`, e.g.:

```go
buf := make([]byte, 100)
copy(buf, "Hello")
```

There are a lot of writers in the standard library that have a WriteString() method which can be used to improve write performance by not requiring an allocation when converting a string to a byte slice. You can take advantage of this optimization by using the io.[WriteString](https://golang.org/pkg/io/#WriteString)() function.

```go
// WriteString writes the contents of the string s to w, which accepts a slice of bytes.
// If w implements a WriteString method, it is invoked directly.
// Otherwise, w.Write is called exactly once.
func WriteString(w Writer, s string) (n int, err error) {
	if sw, ok := w.(stringWriter); ok {
		return sw.WriteString(s)
	}
	return w.Write([]byte(s))
}
```

The function is simple. It first checks if the writer implements a WriteString() method and uses it if available. Otherwise it falls back to copying the string to a byte slice and using the Write() method.

Example :- 

```go
func Handler(w http.ResponseWriter, req *http.Request) {
    io.WriteString(w, "blabla.\n")		
}
```

And:

```go
func Handler(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("blabla\n"))
}
```



---





#### copying bytes

Now that we can read bytes and we can write bytes, it only makes sense that we’d want to plug those two sides together and copy between readers and writers.

##### connecting readers and writers

- ```go
  func Copy(dst Writer, src Reader) (written int64, err error)
  ```

  - uses a 32KB buffer to read from src and then write to dst. This will be issue when our src continues to grow during our copy operation then you’ll end up with more bytes than expected. In this case you can use the [CopyN](https://golang.org/pkg/io/#CopyN)() function to specify an exact number of bytes to be written:

- ```go
  func CopyN(dst Writer, src Reader, n int64) (written int64, err error)
  ```

  - issue with this is it requires an allocation of 32KB buffer on every call, I f you are prforming a lot of copy then you can reuse your own buffer  by using CopyBuffer() instead.

- ```go
  func CopyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error)
  ```
  Let's see the source code of copy function to have a better understanding


```go
func Copy(dst Writer, src Reader) (written int64, err error) {
	return copyBuffer(dst, src, nil)
}

// CopyBuffer is identical to Copy except that it stages through the
// provided buffer (if one is required) rather than allocating a
// temporary one. If buf is nil, one is allocated; otherwise if it has
// zero length, CopyBuffer panics.
func CopyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error) {
	if buf != nil && len(buf) == 0 {
		panic("empty buffer in io.CopyBuffer")
	}
	return copyBuffer(dst, src, buf)
}

// copyBuffer is the actual implementation of Copy and CopyBuffer.
// if buf is nil, one is allocated.
func copyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error) {
	// If the reader has a WriteTo method, use it to do the copy.
	// Avoids an allocation and a copy.
	if wt, ok := src.(WriterTo); ok {
		return wt.WriteTo(dst)
	}
	// Similarly, if the writer has a ReadFrom method, use it to do the copy.
	if rt, ok := dst.(ReaderFrom); ok {
		return rt.ReadFrom(src)
	}
	size := 32 * 1024
	if l, ok := src.(*LimitedReader); ok && int64(size) > l.N {
		if l.N < 1 {
			size = 1
		} else {
			size = int(l.N)
		}
	}
	if buf == nil {
		buf = make([]byte, size)
	}
	for {
		nr, er := src.Read(buf)
		if nr > 0 {
			nw, ew := dst.Write(buf[0:nr])
			if nw > 0 {
				written += int64(nw)
			}
			if ew != nil {
				err = ew
				break
			}
			if nr != nw {
				err = ErrShortWrite
				break
			}
		}
		if er != nil {
			if er != EOF {
				err = er
			}
			break
		}
	}
	return written, err
}

```





##### optimizing copy

- If you want to remove intermediate buffer all together, then your types can implement types interfaces read and write directly. When implemented, the Copy() function will avoid the intermediate buffer and 
  use these implementations directly.

- on write side,

  ```go
  type WriterTo interface {
          WriteTo(w Writer) (n int64, err error)
  }
  ```

- on read side,

  ```go
  type ReaderFrom interface {
          ReadFrom(r Reader) (n int64, err error)
  }
  ```

---



#### adapting readers and writers

Sometimes you’ll find that you have a function that accepts a Reader but all you have is a Writer. Perhaps you need to write out data dynamically to an HTTP request but http.[NewRequest](https://golang.org/pkg/net/http/#NewRequest)() only accepts a Reader.

You can invert a writer by using io.[Pipe](https://golang.org/pkg/io/#Pipe)():

```
func Pipe() (*PipeReader, *PipeWriter)
```

This provides you with a new reader and writer. Any writes to the new PipeWriter will go to the PipeReader.

I rarely use this functionality directly, however, the exec.[Cmd](https://golang.org/pkg/os/exec/#Cmd) uses this for implementing Stdin, Stdout, and Stderr pipes which can be really useful when working with command execution.



---



#### Seeker interface

Streams are usually a continuous flow of bytes from beginning to end but there are a few exceptions. A file, for example, can be operated on as a stream but you can also jump to a specific position within the file.

The [Seeker](https://golang.org/pkg/io/#Seeker) interface is provided to jump around in a stream:

```
type Seeker interface {
        Seek(offset int64, whence int) (int64, error)
}
```

There are 3 ways to jump around: move from on the current position, move from the beginning, and move from the end. You specify the mode of movement using the whence argument. The offset argument specifies how many bytes to move by.

Seeking can be useful if you are using fixed length blocks in a file or if your file contains an index of offsets. Sometimes this data is stored in the header so moving from the beginning makes sense but sometimes this data is specified in a trailer so you’ll need to move from the end.

---



#### Working with individual bytes

- Reading and writing in chunks can be tedious if all you need is a single byte or [rune](https://golang.org/pkg/builtin/#rune). Go provides some interfaces for making this easier.
- The [ByteReader](https://golang.org/pkg/io/#ByteReader) and [ByteWriter](https://golang.org/pkg/io/#ByteWriter) interfaces provide a simple interface for reading and writing single bytes:

```
type ByteReader interface {
        ReadByte() (c byte, err error)
}
```

```
type ByteWriter interface {
        WriteByte(c byte) error
}
```

- You’ll notice that there’s no length arguments since the length will always be either 0 or 1. If a byte is not read or written then an error is returned.
- The [ByteScanner](https://golang.org/pkg/io/#ByteScanner) interface is also provided for working with buffered byte readers:

```
type ByteScanner interface {
        ByteReader
        UnreadByte() error
}
```

- This allows you to push the previously read byte back onto the reader so it can be read the next time. This is particularly useful when writing LL(1) parsers since it allows you to peek at the next available byte.

---



#### Working with individual runes(int32)

- rune is an alias for int32 and is equivalent to int32 in all ways. It is used, by convention, to distinguish character values from integer values. 

- If you are parsing Unicode data then you’ll need to work with runes instead of individual bytes. In that case, the [RuneReader](https://golang.org/pkg/io/#RuneReader) and [RuneScanner](https://golang.org/pkg/io/#RuneScanner) are used instead:

  ```
  type RuneReader interface {
          ReadRune() (r rune, size int, err error)
  }
  type RuneScanner interface {
          RuneReader
          UnreadRune() error
  }
  ```



