[TOC]

# What is encoding exactly?

- In computer science we have fancy words for simple concepts. Not only that but many times there are lots of fancy words for a single concept. **Encoding**  is one of those words. Sometimes it’s referred to as **serialization** or as **marshaling** — it means the same thing: adding a logical structure to raw bytes.
- In the Go standard library, we use the term *encoding and marshaling* for two separate but related ideas. **An *encoder* in Go is an object that applies structure to a stream of bytes while *marshaling* refers to applying structure to bounded, in-memory bytes.**  
  - example 1:-  the *encoding/json* package has a json.[Encoder](https://golang.org/pkg/encoding/json/#Encoder) and json.[Decoder](https://golang.org/pkg/encoding/json/#Decoder) for working with io.[Writer](https://golang.org/pkg/io/#Writer) and io.[Reader](https://golang.org/pkg/io/#Reader) streams, respectively. 
  - The package also has json.[Marshaler](https://golang.org/pkg/encoding/json/#Marshaler) and json.[Unmarshaler](https://golang.org/pkg/encoding/json/#Unmarshaler) for writing to and reading from byte slices.

#### Two types of encoding

- encoding on primitives like strings, integer etc
  - Strings are encoded with character encodings such as ASCII or [Unicode](https://golang.org/pkg/unicode/) or any number of other [language specific encodings](https://godoc.org/golang.org/x/text). 
  - Integers can be encoded differently based on [endianness](https://en.wikipedia.org/wiki/Endianness) or by using variable length encoding. 
  - Even bytes themselves are often encoded using schemes like [Base64](https://golang.org/pkg/encoding/base64/) to convert them into printable characters.
- encoding of objects like structs, maps, slices etc.
  - This refers to converting complex structures such as *structs, maps, and slices* into a series of bytes. There are a lot of tradeoffs when doing this conversion and [many people have developed different object encoding schemes](https://en.wikipedia.org/wiki/Comparison_of_data_serialization_formats) over the years.

#### Making trade-offs ( why to encode and lose speed)

- Converting logical structures to bytes seems simple enough at first —these structures are already represented in-memory as bytes internally. Why not just use that format? Following are the reasons :

  1. Go’s internal data structure format doesn’t match Java’s internal format so we can’t communicate between these different systems.

  2. Sometimes we need compatibility not with another programming language but with humans. [CSV](https://en.wikipedia.org/wiki/Comma-separated_values), [JSON](http://www.json.org/), and [XML](https://en.wikipedia.org/wiki/XML) are all human-readable formats that can be easily viewed and edited.

  3. Our data structures change over time but we still need to operate on bytes that may have been encoded years ago. Some encodings, such as [Protocol Buffers](https://developers.google.com/protocol-buffers/), allow you to write a schema for your data and version your fields — older fields can be deprecated while new fields can be added. The downside of this is that you need the schema definition in order to encode and decode objects. Go’s own [gob](https://golang.org/pkg/encoding/gob/) format takes a different approach and actually includes the schema format when encoding. However, the downside of this approach is that the encoded size can be much larger.

     > Some formats throw caution to the wind entirely and go schema-less. [JSON](http://www.json.org/) and [MessagePack](http://msgpack.org/index.html) both allow you to encode structures on the fly but provide no guarantees about safely decoding structures from an older format.

We also use systems that do encoding for us but we don’t think of as encoding. Databases, for example, are a roundabout way of taking our logical data structures and eventually persisting them as bytes on disk. It may involve network calls, SQL parsing, and query planning but it’s all essentially encoding.

Finally, if you really need speed above all else, you could use Go’s internal format to save data. I even wrote a library for this called [raw](https://github.com/boltdb/raw). It’s encoding and decoding time is literally zero seconds. Should you use it in production? Probably not.

### The 4 interfaces of encoding

- If you are one of the few people who has ever looked at the [encoding](https://golang.org/pkg/encoding/) package, you may have been underwhelmed. It is the second smallest package after the [errors](https://golang.org/pkg/errors/) package and it only includes 4 interfaces.
- The first two interfaces are [BinaryMarshaler](https://golang.org/pkg/encoding/#BinaryMarshaler) and [BinaryUnmarshaler](https://golang.org/pkg/encoding/#BinaryUnmarshaler):

```
type BinaryMarshaler interface {
        MarshalBinary() (data []byte, err error)
}
```

```
type BinaryUnmarshaler interface {
        UnmarshalBinary(data []byte) error
}
```

These are for objects that provide a way to convert to and from a binary format. This is used in a few spots in the standard library such as time.[Time](https://golang.org/pkg/time/#Time).[MarshalBinary](https://golang.org/pkg/time/#Time.MarshalBinary)(). You don’t find it more places because there’s not usually a single defined way to marshal an object to binary format. As we’ve seen, there are a multitude of serialization formats.

At the application level, however, you have probably picked a single format for marshaling. For instance, you may have chosen Protocol Buffers for all your data. There’s is typically no reason to support multiple binary formats for your application data so implementing [BinaryMarshaler](https://golang.org/pkg/encoding/#BinaryMarshaler) can make sense.

- The next two interfaces are [TextMarshaler](https://golang.org/pkg/encoding/#TextMarshaler) and [TextUnmarshaler](https://golang.org/pkg/encoding/#TextUnmarshaler):

```
type TextMarshaler interface {
        MarshalText() (text []byte, err error)
}
```

```
type TextUnmarshaler interface {
        UnmarshalText(text []byte) error
}
```

These two interfaces are similar to the binary marshaling interfaces except that they output in a UTF-8 format.

Some formats have their own marshaling interfaces, such as json.[Marshaler](https://golang.org/pkg/encoding/json/#Marshaler), which follow the same naming style.

### Overview of encoding packages

There are a lot of useful encoding packages baked into the standard library. We’ll cover these in more detail in future posts but I’d like to give an overview first. Some of these are subpackages of *encoding* while others are scattered in different locations.

#### Primitive encodings

###### String Encoding

- The first package you probably used when you started with Go is the [fmt](https://golang.org/pkg/fmt/) package (pronounced “fumpt”). It uses C-style [printf](http://pubs.opengroup.org/onlinepubs/009695399/functions/fprintf.html)() conventions to encode and decode numbers, strings, bytes, and even includes limited support for object encoding. The [fmt](https://golang.org/pkg/fmt/) package is a great, simple way to build human-readable strings from templates but the template parsing can add overhead.
- If you need better performance then you can avoid templating by using the string conversion package — [strconv](https://golang.org/pkg/strconv/). This low-level package provides basic formatting and scanning for strings, integers, floats, and booleans and is generally pretty fast.
- These packages, along with Go itself, assume that you’re encoding strings using UTF-8. The near total lack of non-Unicode character encoding support in the standard library could be because the Internet has quickly converged on a standard of UTF-8 over the last several years or it could be because [Rob Pike](https://en.wikipedia.org/wiki/Rob_Pike) is a coauthor of Go & UTF-8. Who knows? I’ve been lucky enough to not have to deal with any non UTF-8 encodings in Go so far, however, there is some encoding support in [unicode/utf16](https://golang.org/pkg/unicode/utf16/), [encoding/ascii85](https://golang.org/pkg/encoding/ascii85/), and the [golang.org/x/text](https://godoc.org/golang.org/x/text) package tree. The *“x”* package tree contains a wealth of awesome packages that are part of the Go project but are not covered under the [Go 1 compatibility requirements](https://golang.org/doc/go1compat).

###### Integer Encoding

- For integer encoding, the [encoding/binary](https://golang.org/pkg/encoding/binary/) package provides big endian and little endian encodings as well as variable length encodings. Endianness refers to the order that bytes are written to disk. For example, the *uint16* representation of *1,000* (which is *0x03E8* in hex) is composed of 2 bytes: *03* & *E8*. With big endian encoding, the bytes are written in that order “03 E8”. In little endian, the order is reversed: “E8 03”. Many common CPUs architectures use little endian. **However, big endian is typically used when sending bytes over the network. Big endian is even called *network byte order*.**

###### Byte encoding

Finally, for byte encoding there are a couple packages available. Byte encoding is typically used to convert bytes into a printable format. The [encoding/hex](https://golang.org/pkg/encoding/hex/) package, for example, can be used if you need to view binary data in hexidecimal format. I’ve personally only used it for debugging purposes. On the other hand, sometimes you need a printable format because you need to transport data over protocols with historically limited binary support (such as email). The [encoding/base32](https://golang.org/pkg/encoding/base32/) and [encoding/base64](https://golang.org/pkg/encoding/base64/) packages are an example of this. Another example is the [encoding/pem](https://golang.org/pkg/encoding/pem/) package which is used for encoding TLS certificates.

> **base64 ??**
>
> When you have some binary data that you want to ship across a network, you generally don't do it by just streaming the bits and bytes over the wire in a raw format. Why? because some media are made for streaming text. You never know -- some protocols may interpret your binary data as control characters (like a modem), or your binary data could be screwed up because the underlying protocol might think that you've entered a special character combination (like how FTP translates line endings).
>
> So to get around this, people encode the binary data into characters. Base64 is one of these types of encodings. Infact int theory we can do base-80 or something similar, but it would be siginificantly harder. Powers of 2 are natural bases for binary.
>
> **Why 64?**
> Because you can generally rely on the same 64 characters being present in many character sets, and you can be reasonably confident that your data's going to end up on the other side of the wire uncorrupted.

#### Object encodings

-  There are only few packages for byte encoding in go standard library.

  1. **encoding/json package** 

      JSON has its flaws but it’s easy to use and it has library support in every language so adoption has skyrocketed. The [encoding/json](https://golang.org/pkg/encoding/json/) package provides great support for this protocol and there are also third party implementations for generating faster parsers such as [ffjson](https://github.com/pquerna/ffjson).

  2. **encoding/csv package**

     While JSON has dominated as a protocol between machines, the [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) format is a more common protocol for exporting data to humans. The [encoding/csv](https://golang.org/pkg/encoding/csv/) package provides a good interface for exporting tabular data in this format.

  3. **encoding/xml package**

     The [encoding/xml](https://golang.org/pkg/encoding/xml/) package provides a [SAX](https://en.wikipedia.org/wiki/Simple_API_for_XML)-style interface with an additional tag-driven marshaler/unmarshaler that’s similar to the [json](https://golang.org/pkg/encoding/json/) package. If you’re looking for more complex features like DOM, XPath, XSD, or XSLT then you should probably use [libxml2](http://xmlsoft.org/) via cgo.

  4. **gob package**

     Go has its own stream encoding called gob. This package is ussed by the net/rpc package for implementing a RPC call between two go services. It does not have cross-language support, use grpc instead that seems to be a popular alternative.

  5. **encoding/asn1**

     ASN.1 is a complex object encoding scheme that is most notably used by X.509 certificates in SSL/TLS.



# Advanced Encoding and Decoding Techniques

Here we'll look at :-

1. encoding json (text-based protocols)
2. encoding binary (binary-protocols , they can be much better fit when two machines need to communicate quickly and effeciently)



## Encoding json



Go’s standard library comes packed with some great encoding and decoding packages covering a wide array of encoding schemes. Everything from CSV, XML, JSON, and even gob - a Go specific encoding format - is covered, and all of these packages are incredibly easy to get started with. In fact, most of them don’t require you to add any code at all; You simply plug in your data and it spits out encoded data.

Unfortunately, not all applications have the pleasure of working with data that maps one-to-one to its JSON representation. Struct tags help cover most of these use cases, but if you work with enough APIs you are bound to eventually come across a case where it just isn’t enough.

For example, you might run into an API that outputs different objects to the same key, making it a prime candidate for generics, but Go doesn’t have those. Or you might be using an API that accepts and returns [Unix time](https://en.wikipedia.org/wiki/Unix_time) instead of RFC 3339; Sure, we could leave it as an int in our code, but it would be much nicer if we could work directly with the [time](https://golang.org/pkg/time/) package’s [Time](https://golang.org/pkg/time/#Time) type.

In this post we are going to review a few techniques that help turn what might seem like a troublesome encoding problem into some fairly easy to follow code. We will be doing this using the [encoding/json](https://golang.org/pkg/encoding/json/) package, but it is worth noting that Go provides a `Marshaler` and `Unmarshaler` interface for most encoding types, allowing you to customize how your data is encoded and decoded in multiple encoding schemes.

### The always-faithful new type approach

The first technique we are going to examine is to create a new type and convert our data to/from that type before encoding and decoding it. While this isn’t technically an encoding specific solution, it is one that works reliably, and is fairly easy to follow. It is also a basic technique that we will be building upon in the next few sections, so it is worth taking some time to look at.

Let’s imagine that our application is starting off with a simple `Dog` type like below.

```
type Dog struct {
  ID      int
  Name    string
  Breed   string
  BornAt  time.Time
}

```

By default, the [time.Time](https://golang.org/pkg/time/#Time) type will be marshaled as RFC 3339. That is, it will turn into a string that looks something like `2016-12-07T17:47:35.099008045-05:00`.

While there is nothing particularly wrong with this format, we might have a reason to want to encode and decode this field differently. For example, we might be working with an API that sends us a Unix time and expects the same in return.

Regardless of the reason, we need a way to change how this is turned into JSON, along with how it is parsed from JSON. One approach to solving this problem is to simply create a new type, let’s call it the `JSONDog` type, and use that to encode and decode our JSON.

```
type JSONDog struct {
  ID     int    `json:"id"`
  Name   string `json:"name"`
  Breed  string `json:"breed"`
  BornAt int64  `json:"born_at"`
}

```

Now if we want to encode our original `Dog` type in JSON, all we need to do is convert it into a `JSONDog` and then marshal that using the `encoding/json` package.

Starting with a constructor that takes in a `Dog` and returns a `JSONDog` we get the following code.

```
func NewJSONDog(dog Dog) JSONDog {
  return JSONDog{
    dog.ID,
    dog.Name,
    dog.Breed,
    dog.BornAt.Unix(),
  }
}

```

Putting that together with the `encoding/json` package we get the following sample.

```
func main() {
  dog := Dog{1, "bowser", "husky", time.Now()}
  b, err := json.Marshal(NewJSONDog(dog))
  if err != nil {
    panic(err)
  }
  fmt.Println(string(b))
}

```

Decoding a JSON object into a `Dog` works very similar to this. We first start by decoding into a `JSONDog` instance, and then we convert that back to our `Dog` type using a `Dog()` method on the `JSONDog` type.

```
func (jd JSONDog) Dog() Dog {
  return Dog{
    jd.ID,
    jd.Name,
    jd.Breed,
    time.Unix(jd.BornAt, 0),
  }
}

func main() {
  b := []byte(`{
    "id":1,
    "name":"bowser",
    "breed":"husky",
    "born_at":1480979203}`)
  var jsonDog JSONDog
  json.Unmarshal(b, &jsonDog)
  fmt.Println(jsonDog.Dog())
}

```

*You can see the full code sample and run it on the Go Playground here: https://play.golang.org/p/0hEhCL0ltW*

The primary benefit to this approach is that it will always work because we can always build that translation layer. It doesn’t matter if the JSON representation looks nothing like our Go code, so long as we have a way to convert between the two.

In addition to always working, the code is incredibly easy to follow. A new teammate could jump right into this code without missing a beat because there isn’t any magic happening. We are just converting data between two types.

Despite the benefits of this technique, there are a few cons. The two biggest are:

1. It is easy for a developer to forget to convert a `Dog` into a `JSONDog`, and
2. It takes a lot of extra code, especially if we have a large struct and only a couple fields need customized

Rather than throw this technique out the window, let’s take a look at how to tackle both of these problems without sacrificing much code clarity.

### Implementing the `Marshaler` and `Unmarshaler` interfaces

The first problem we saw with the last approach is that it is pretty easy to forget to convert a `Dog` into a `JSONDog`, so in this section we are going to discuss how you can implement the [Marshaler](https://golang.org/pkg/encoding/json/#Marshaler) and [Unmarshaler](https://golang.org/pkg/encoding/json/#Unmarshaler) interfaces in the `encoding/json` package to make the conversion automatic.

The way these two interfaces work is pretty straight forward; When the `encoding/json` package runs into a type that implements the `Marshaler` interface, it uses that type’s `MarshalJSON()` method instead of the default marshaling code to turn the object into JSON. Similarly, when decoding a JSON object it will test to see if the object implements the `Unmarshaler` interface, and if so it will use the `UnmarshalJSON()` method instead of the default unmarshaling behavior. That means that all we need to do to ensure our `Dog` type is always encoded and decoded as a `JSONDog` is to implement these two methods and do the conversion there.

Once again, we are going to start with encoding first, which means we will be implementing the `MarshalJSON() ([]byte, error)` method on our `Dog` type.

While this might feel like a large undertaking at first, we can actually utilize a lot of our existing code to minimize what we need to write. All we really need to do in this method is return the results from calling `json.Marshal()` on a `JSONDog` representation of our current `Dog`.

```
func (d Dog) MarshalJSON() ([]byte, error) {
  return json.Marshal(NewJSONDog(d))
}

```

Now it doesn’t matter if a developer forgets to convert our `Dog` type to the `JSONDog` type; This will happen by default when the `Dog` is encoded into JSON.

The `Unmarshaler` implementation ends up being pretty similar. We are going to implement the `UnmarshalJSON([]byte) error` method, and once again we are going to utilize our existing `JSONDog` type.

```
func (d *Dog) UnmarshalJSON(data []byte) error {
  var jd JSONDog
  if err := json.Unmarshal(data, &jd); err != nil {
    return err
  }
  *d = jd.Dog()
  return nil
}

```

Finally, we update our `main()` function to use the `Dog` type for both decoding and encoding instead of the `JSONDog` type.

```
func main() {
  dog := Dog{1, "bowser", "husky", time.Now()}
  b, err := json.Marshal(dog)
  if err != nil {
    panic(err)
  }
  fmt.Println(string(b))

  b = []byte(`{
    "id":1,
    "name":"bowser",
    "breed":"husky",
    "born_at":1480979203}`)
  dog = Dog{}
  json.Unmarshal(b, &dog)
  fmt.Println(dog)
}

```

*You can find a working sample of the code up until this point on the Go Playground here: https://play.golang.org/p/GR6ckydMxF*

With about 10 lines of code we have overridden the default JSON encoding for our `Dog` type. Pretty neat, right?

Next we will tackle the other problem we had with our initial approach using embedded data and an alias type.

### Using embedded data and an alias type to reduce code

*NOTE: The term “alias” here is NOT the same as the alias proposed for Go 1.9. This is simply referring to a new type that has the same data as another type, but has its own set of methods.*

As we saw before, copying all of the data from one type to another just to customize a single field can be pretty tedious. This is amplified even further when we start to work with objects with 10 or 20 fields. Keeping both a `JSONDog` and a `Dog` type in sync is just annoying to do.

Luckily, there is another way to tackle this problem that will reduce the fields we need to customize to just those that need custom encoding and decoding. We are going to embed our `Dog` object inside of our `JSONDog` and customize the fields that need customizing.

To get started, we first need to update the `Dog` type and add the JSON struct tags back for fields we won’t be customizing, and we will tell the `encoding/json` package to ignore fields we will be customizing by using the struct tag `json:"-"` which signifies that the JSON encoder should ignore this field even though it is exported.

```
type Dog struct {
  ID     int       `json:"id"`
  Name   string    `json:"name"`
  Breed  string    `json:"breed"`
  BornAt time.Time `json:"-"`
}

```

Next, we are going to embed the `Dog` type inside of our `JSONDog` type, and update the `NewJSONDog()` function along with the `Dog()` method on the `JSONDog` type. We will also be temporarily renaming the `Dog()` method to `ToDog()` so it doesn’t collide with the embedded `Dog` object.

*Warning: This code will not work just yet, but I am showing this intermediate step to illustrate why it won’t work.*

```
func NewJSONDog(dog Dog) JSONDog {
  return JSONDog{
    dog,
    dog.BornAt.Unix(),
  }
}

type JSONDog struct {
  Dog
  BornAt int64 `json:"born_at"`
}

func (jd JSONDog) ToDog() Dog {
  return Dog{
    jd.Dog.ID,
    jd.Dog.Name,
    jd.Dog.Breed,
    time.Unix(jd.BornAt, 0),
  }
}

```

Unfortunately, this code won’t work. It will compile, but if you try to run it you will run into a `fatal error: stack overflow` error. This happens when we call the `MarshalJSON()` method on the `Dog` type. When this happens, it will construct a `JSONDog`, but this object has a nested `Dog` inside of it, which will construct a new `JSONDog`, and this cycle will repeat infinitely until our application crashes.

To avoid this, we need to create an alias dog type that doesn’t have the `MarshalJSON()` and `UnmarsshalJSON()` methods.

```
type DogAlias Dog

```

Once we have our alias type, we can update our `JSONDog` type to embed this instead of the `Dog` type. We will also need to update our `NewJSONDog()` function to convert our `Dog` into a `DogAlias`, and we can clean up our `Dog()` method on the `JSONDog` type a bit by utilizing the nested `Dog` as our return value.

```
func NewJSONDog(dog Dog) JSONDog {
  return JSONDog{
    DogAlias(dog),
    dog.BornAt.Unix(),
  }
}

type JSONDog struct {
  DogAlias
  BornAt int64 `json:"born_at"`
}

func (jd JSONDog) Dog() Dog {
  dog := Dog(jd.DogAlias)
  dog.BornAt = time.Unix(jd.BornAt, 0)
  return dog
}

```

As you can see, the initial setup for this took around 30 lines of code, but now that we have it set up, it really doesn’t matter how many fields are in the `Dog` type. Our JSON code will only grow if we increase the number of fields with custom JSON.

*You can find all of the code from this section on the Go Playground here: https://play.golang.org/p/N0rweY-cD0*

### Custom types for specific fields

The approach we looked at in the last section focused heavily on converting our entire object into another type before encoding and decoding it, but even with the embedded alias, we would need to repeat this code for every different type of object that has a `time.Time` field.

In this section we are going to be looking at an approach that allows us to define how we want a type to encode and decode just once, and then we will reuse that type across our application. Going back to our original example, we are again going to start with the `Dog` type with a `BornAt` field that needs custom JSON.

```
type Dog struct {
  ID     int       `json:"id"`
  Name   string    `json:"name"`
  Breed  string    `json:"breed"`
  BornAt time.Time `json:"born_at"`
}

```

We already know that this won’t work, so rather than using the `time.Time` type, we are going to create our own `Time` type, embed the `time.Time` inside of the new type, and then update our `Dog` type to use our new `Time` type.

```
type Dog struct {
  ID     int    `json:"id"`
  Name   string `json:"name"`
  Breed  string `json:"breed"`
  BornAt Time   `json:"born_at"`
}

type Time struct {
  time.Time
}

```

Next, we are going to write a custom `MarshalJSON()` and `UnmarshalJSON()` method for our `Time` type. These will output a Unix time, and parse a Unix time respectively.

```
func (t Time) MarshalJSON() ([]byte, error) {
  return json.Marshal(t.Time.Unix())
}

func (t *Time) UnmarshalJSON(data []byte) error {
  var i int64
  if err := json.Unmarshal(data, &i); err != nil {
    return err
  }
  t.Time = time.Unix(i, 0)
  return nil
}

```

And that’s it! We can now freely use our new `Time` type in all of our structs and it will be encoded and decoded as a Unix time. On top of that, because we embedded the `time.Time` object, we can even freely use methods like `Day()` on our own `Time` type, meaning a good portion of our code won’t need to be rewritten.

That does bring up one of the downsides to this approach; By using a new type, we do stand the chance of breaking some of our code that expects the `time.Time` type rather than our new `Time` type. You can update all of your code to use the new type, or you can even access the embedded `time.Time` object, so it isn’t impossible to fix this, but it may require a little bit of refactoring.

Another solution to this problem is to merge this approach with the first one we looked at. That way we get the best of both worlds - our `Dog` type has a `time.Time` object, but our `JSONDog` doesn’t need to worry itself with as many specifics for converting between the two types; The conversion logic is all contained within the new `Time` type.

*A full example of this can be seen on the Go Playground here: https://play.golang.org/p/C272eojwTh*

### Encoding and decoding generics

The last technique we are going to look at is a little different than the first two we examined, and is meant to solve an entirely different problem - the problem of dynamic types being stored in nested JSON.

For example, imagine you could get the following JSON responses from a server:

```
{
  "data": {
    "object": "bank_account",
    "id": "ba_123",
    "routing_number": "110000000"
  }
}

```

And from the same endpoint you might also receive the following:

```
{
  "data": {
    "object": "card",
    "id": "card_123",
    "last4": "4242"
  }
}

```

At first glance these might look like similar responses with optional data, but they are in fact completely different objects. What you can do with a bank account is different from what you can do with a card, and while I haven’t shown them all here, each of these would likely have very different fields.

One solution to this problem would be to use generics, and to set the type of the nested data when decoding the JSON. You have to use the reflect library a bit, but it *is possible* in a language like Java, and you would end up with some classes like those below.

```
class Data<T> {
  public T t;
}
class Card {...}
class BankAccount {...}

```

Go, on the other hand, doesn’t have generics, so how are we supposed to parse this JSON?

One option is to use a map with our keys being strings, but what data type should we use for our values? Even if we assume it is a nested map, what happens if the card object has an integer value, or a nested JSON object inside of it?

That really limits our options, and we are essentially stuck using the empty interface (`interface{}`).

```
func main() {
  jsonStr := `
{
  "data": {
    "object": "card",
    "id": "card_123",
    "last4": "4242"
  }
}
`
  var m map[string]map[string]interface{}
  if err := json.Unmarshal([]byte(jsonStr), &m); err != nil {
    panic(err)
  }
  fmt.Println(m)

  b, err := json.Marshal(m)
  if err != nil {
    panic(err)
  }
  fmt.Println(string(b))
}

```

Using the empty interface type comes with its own unique set of problems, the most notable being that the empty interface literally tells us nothing about the data. If we want to know anything about the data stored at any key, we are going to need to do a type assertion, and that sucks. Thankfully, there are other ways to approach this problem!

This approach is going to once again take advantage of the `Marshaler` and `Unmarshaler` interfaces, but this time we are going to add a bit of conditional logic to our code, and we are going to use a type that has both a pointer to a `Card`, and a pointer to a `BankAccount` in it. When we start to decode our JSON, we will first decode the `object` key to determine which of these two fields we should fill, and then we will fill the corresponding field.

We will start by declaring our types. The `BankAccount` and `Card` types are pretty easy - we are mapping the JSON direct to a Go struct.

```
type BankAccount struct {
  ID            string `json:"id"`
  Object        string `json:"object"`
  RoutingNumber string `json:"routing_number"`
}

type Card struct {
  ID     string `json:"id"`
  Object string `json:"object"`
  Last4  string `json:"last4"`
}

```

Next we have our `Data` type. You are welcome to name this whatever you want, and it might make sense to use something like `Source` or `CardOrBankAccount`, but that tends to vary from case to case, so I am sticking with `Data` for now.

```
type Data struct {
  *Card
  *BankAccount
}

```

*We are using pointers here because we won’t ever be initializing both of these. Instead, we will determine which one needs to be used, and initialize that one. That means in your code that does use this type, you might need to write something like if data.Card != nil { ... } to determine if it is a card. Alternatively, you could also store the Object attribute on the Data type itself. That choice is ultimately up to you, and it may require some minor code tweaks, but the overall approach discussed here should still work.*

Now that we have a `Data` type, we are going to update our `main()` function so it is clear how this object will map to our JSON.

```
func main() {
  jsonStr := `
{
  "data": {
    "object": "card",
    "id": "card_123",
    "last4": "4242"
  }
}
`
  var m map[string]Data
  if err := json.Unmarshal([]byte(jsonStr), &m); err != nil {
    panic(err)
  }
  fmt.Println(m)
  data := m["data"]
  if data.Card != nil {
    fmt.Println(data.Card)
  }
  if data.BankAccount != nil {
    fmt.Println(data.BankAccount)
  }

  b, err := json.Marshal(m)
  if err != nil {
    panic(err)
  }
  fmt.Println(string(b))
}

```

`Data` isn’t meant to be the entire JSON structure, but is instead meant to represent everything stored inside the `data` key in the JSON object. This is important to note, because in our code our `Data` type has both `Card` and `BankAccount` pointers, but in the JSON these won’t be nested objects. That means the first thing we need to do is write a `MarshalJSON()` method to reflect this.

```
func (d Data) MarshalJSON() ([]byte, error) {
  if d.Card != nil {
    return json.Marshal(d.Card)
  } else if d.BankAccount != nil {
    return json.Marshal(d.BankAccount)
  } else {
    return json.Marshal(nil)
  }
}

```

This code is first checking to see if we have a `Card` or `BankAccount` object. If either is present, it will output the JSON for that corresponding object. If neither is present, it will instead output the JSON for `nil`, which is `null` in JSON.

*The last bit of the MarshalJSON() method might vary in your own code. You may want to return an error in cases like this, or you may want to encode an empty map, but for this example we are using nil.*

`MarshalJSON()` wasn’t a big departure from what we have done so far, but the `UnmarshalJSON()` method is going to be a bit different. In this method we are going to end up parsing the data twice. The first time we are going to parse using a struct that only has an `Object` field simply so we can determine what type we should be using to decode our JSON, and then we will use that type to decode the JSON.

```
func (d *Data) UnmarshalJSON(data []byte) error {
  temp := struct {
    Object string `json:"object"`
  }{}
  if err := json.Unmarshal(data, &temp); err != nil {
    return err
  }
  if temp.Object == "card" {
    var c Card
    if err := json.Unmarshal(data, &c); err != nil {
      return err
    }
    d.Card = &c
    d.BankAccount = nil
  } else if temp.Object == "bank_account" {
    var ba BankAccount
    if err := json.Unmarshal(data, &ba); err != nil {
      return err
    }
    d.BankAccount = &ba
    d.Card = nil
  } else {
    return errors.New("Invalid object value")
  }
  return nil
}

```

There are also a few other minor things going on here, like setting the unused field to nil after decoding the data. I am doing this to ensure that our `Data` object is cleared of old data, and our encoding code doesn’t run into any bugs. This works well in this case because it isn’t ever really valid to have both a `Card` and a `BankAccount` in our `Data` type; Only one should ever be set.

I also define the struct with the `Object` field dynamically, but you could just as easily declare the type elsewhere and use it here.

*You can find a runnable version of the code from this section on the Go Playground here: https://play.golang.org/p/gLAgLQv9Et*



## Encoding Binary

- Writing binary protocols seems a daunting task at beggining, but it's surprisingly simple if you grasp some key concepts.



### What is Binary protocol?

- The distinction between text protocols and binary protocols may seem silly since they both communicate using bytes. However, the important distinction is in regards to which bytes they can use and how they structure them.
- Text protocols, like JSON, use only the printable set of characters in ASCII or Unicode to communicate. For example, the number “26” is represented using the “2” and “6” bytes because those are printable characters. This is great for humans to read but slow for computers to read.
- With a binary protocol, the number “26” can be represented using a single byte — *0x1A* in hexadecimal. That’s a 50% reduction in space and it’s already in the computer’s native binary format so it doesn’t need to be parsed. This performance difference looks insignificant for a single number but it adds up when processing millions or billions of numbers.

### This wierd thing called 'endianess'

- Big endian is when you write your *most significant byte* first and little endian is when you write your *least significant byte* first.

  eg - *287,454,020* which is ***0x11223344*** in hexidecimal. The most significant byte is *0x11* and the least significant byte is *0x44*.

  Encoding this in big endian looks like:

  ```
  11 22 33 44
  ```

  and encoding in little endian looks like:

  ```
  44 33 22 11
  ```

- **Big endian also has an interesting property that it is *lexicographically sortable***.That means that you can compare two binary-encoded numbers starting from the first byte and moving to the last byte. That’s how bytes.[Equal](https://golang.org/pkg/bytes/#Equal)() and bytes.[Compare](https://golang.org/pkg/bytes/#Compare)() work. This is because the most significant bytes come first in big endian encoding.

#### So why would you ever want to use little endian?      

- The benefit to this seemingly backwards approach is that you can change the size of your number type without moving bytes.

- For example, we can easily change this 4-byte *int32* number we have above to an 8-byte *int64* number by simply increasing the length with four zero-padded bytes. This makes it so the *int32* and *int64* point to the same memory address:

  ```
  44 33 22 11
  44 33 22 11 00 00 00 00
  ```

  This kind of operation is useful at the compiler level to extend and shrink integer variables but not terribly useful at the protocol level.

### What is Network byte order ?

- Big endian encoding is the convention when operating on binary numbers over a network protocol — so much so that it’s actually referred to as network byte order.





