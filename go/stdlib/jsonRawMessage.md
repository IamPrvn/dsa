- [AboutAbout me and this site](http://eagain.net/about/)
- [BlogA journal of my thoughts and things in my life](http://eagain.net/blog/)
- [ArticlesCollection of articles I've published](http://eagain.net/articles/)
- [TalksPresentations I have delivered recently](http://eagain.net/talks/)
- [SoftwareSoftware I have written](http://eagain.net/software/)
- [LinksMy link collection](http://eagain.net/links/)
- [PhotosMy picture collection, featuring the cute little chihuahua Pico](http://eagain.net/cam/)

# Dynamic JSON in Go

Go is a statically typed language. While it can represent dynamictypes, making a nested `map[string]interface{}` duck quack leads tovery ugly code. We can do better, by embracing the static nature ofthe language.

The need for dynamic, or more appropriately *parametric*, content inJSON often arises in situations where there's multiple kinds ofmessages being exchanged over the same communication channel. First,let's talk about *message envelopes*, where the JSON looks like this:

```
{
	"type": "this part tells you how to interpret the message",
	"msg": ...the actual message is here, in some kind of json...
}

```

## Generating JSON with different message types

Marshaling data structures into JSON with multiple types of messagebodies with a separate envelope is easy with `interface{}`. Togenerate this:

```
{
	"type": "sound",
	"msg": {
		"description": "dynamite",
		"authority": "the Bruce Dickinson"
	}
}

```

```
{
	"type": "cowbell",
	"msg": {
		"more": true
	}
}

```

We can use these Go types:

```
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

type Envelope struct {
	Type string
	Msg  interface{}
}

type Sound struct {
	Description string
	Authority   string
}

type Cowbell struct {
	More bool
}

func main() {
	s := Envelope{
		Type: "sound",
		Msg: Sound{
			Description: "dynamite",
			Authority:   "the Bruce Dickinson",
		},
	}
	buf, err := json.Marshal(s)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s\n", buf)

	c := Envelope{
		Type: "cowbell",
		Msg: Cowbell{
			More: true,
		},
	}
	buf, err = json.Marshal(c)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s\n", buf)
}

```

Output:

```
{"Type":"sound","Msg":{"Description":"dynamite","Authority":"the Bruce Dickinson"}}
{"Type":"cowbell","Msg":{"More":true}}

```

Nothing special there.

## Unmarshal into dynamic hell

If you just ask to unmarshal the above JSON objects into the`Envelope` type from above, you'll end with `Msg` being a`map[string]interface{}`. That's not very nice to use, and will makeyou regret your life choices:

```
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

const input = `
{
	"type": "sound",
	"msg": {
		"description": "dynamite",
		"authority": "the Bruce Dickinson"
	}
}
`

type Envelope struct {
	Type string
	Msg  interface{}
}

func main() {
	var env Envelope
	if err := json.Unmarshal([]byte(input), &env); err != nil {
		log.Fatal(err)
	}
	// for the love of Gopher DO NOT DO THIS
	var desc string = env.Msg.(map[string]interface{})["description"].(string)
	fmt.Println(desc)
}

```

Output:

```
dynamite

```

## Unmarshaling the very explicit way

Previously, I used to recommend changing the `Envelope` type, likethis:

```
type Envelope {
	Type string
	Msg  *json.RawMessage
}

```

[`json.RawMessage`](http://golang.org/pkg/encoding/json/#RawMessage)is a useful type that lets you postpone the unmarshaling; it juststores the raw data as a `[]byte`.

This lets you explicitly control the unmarshaling of `Msg`, and thusdelay that until after you've branched out based on the value of`Type`.

The downside of that is that now you need to explicitly marshal `Msg`first, or you need separate `EnvelopeIn` and `EnvelopeOut` types,where `EnvelopeOut` still has `Msg interface{}`.

We can do better than that.

## Combining the powers of `*json.RawMessage` and `interface{}`

So, how to combine the good aspects of the two approaches above? Byputting a `*json.RawMessage` in the `interface{}` field!

```
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

const input = `
{
	"type": "sound",
	"msg": {
		"description": "dynamite",
		"authority": "the Bruce Dickinson"
	}
}
`

type Envelope struct {
	Type string
	Msg  interface{}
}

type Sound struct {
	Description string
	Authority   string
}

func main() {
	var msg json.RawMessage
	env := Envelope{
		Msg: &msg,
	}
	if err := json.Unmarshal([]byte(input), &env); err != nil {
		log.Fatal(err)
	}
	switch env.Type {
	case "sound":
		var s Sound
		if err := json.Unmarshal(msg, &s); err != nil {
			log.Fatal(err)
		}
		var desc string = s.Description
		fmt.Println(desc)
	default:
		log.Fatalf("unknown message type: %q", env.Type)
	}
}

```

Output:

```
dynamite

```

## How to put everything at the top level

While I heartily recommend you put the parametric body under a singlekey, sometimes you work with pre-existing formats that just don't dothat.

Please use the earlier style if you can.

```
{
	"type": "this part tells you how to interpret the message",
	...the actual message is here, as multiple keys...
}

```

We can cope with that by unmarshaling the data twice:

```
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

const input = `
{
	"type": "sound",
	"description": "dynamite",
	"authority": "the Bruce Dickinson"
}
`

type Envelope struct {
	Type string
}

type Sound struct {
	Description string
	Authority   string
}

func main() {
	var env Envelope
	buf := []byte(input)
	if err := json.Unmarshal(buf, &env); err != nil {
		log.Fatal(err)
	}
	switch env.Type {
	case "sound":
		var s struct {
			Envelope
			Sound
		}
		if err := json.Unmarshal(buf, &s); err != nil {
			log.Fatal(err)
		}
		var desc string = s.Description
		fmt.Println(desc)
	default:
		log.Fatalf("unknown message type: %q", env.Type)
	}
}

```

```
dynamite

```

## Parting thoughts

Hope this inspired you to clean up your JSON handling. Also see thefollow-up article for[how to avoid the `switch` statements you see here](http://eagain.net/articles/go-json-kind/).

I have more related topics in store, but please let me know what youfind interesting! You can reach me at[tv@eagain.net](mailto:tv@eagain.net).

------