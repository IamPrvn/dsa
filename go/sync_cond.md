[TOC]



# Understanding Condition Variable in Go

If you had ever done some kind of multi-threaded or concurrent programming in any languages, then there is a high chance that you would have used or at least heard about Condition Variable.

This post is an attempt to explain my understanding of the Condition Variable, which I happened to use in one of my recent projects. All the code samples are written in Go, but the concepts remains same irrespective of the programming languages.

> A condition variable is basically a container of threads that are waiting for a certain condition.

That definition from wikipedia is pretty straight forward. And don’t worry if that still confusing. I hope this blog may help you understand what it is and how it can be used.

Condition Variable is one of the synchronization mechanisms.

Before using Condition Variable, lets understand what problem it tries to solve.

Lets start simple.

> How to make a goroutine wait till some event(condition) occur?.

Lets say the event be waiting for non-zero value in a struct field.

## Naive way: Use infinite for loop.

```
type Record struct {
	sync.Mutex
	data string
}

func main() {
	var wg sync.WaitGroup

	rec := &Record{}
	wg.Add(1)
	go func(rec *Record) {
		defer wg.Done()
		for {
			rec.Lock()
			if rec.data != "" {
				fmt.Println("Data: ", rec.data)
				rec.Unlock()
				return
			}
			rec.Unlock()
		}
	}(rec)

	time.Sleep(2 * time.Second)
	rec.Lock()
	rec.data = "gopher"
	rec.Unlock()

	wg.Wait() // wait till all goutine completes
}

```

Though above approach would work, the goroutine keeps iterating in the for loop just by wasting the CPU cycles.

We could use `time.Sleep`(Go runtime scheduler suspends a sleeping goroutine) but that wouldn’t be a right approach (say, how long the goroutine have to sleep?).

The problem here is we need some way to make goroutine suspend while waiting(so that CPU cycles are not wasted) and some way to *signal* the suspended goroutine when particular event has occured (say when `data` field is non-empty in this case).

Enter **Condition** **Variable**.

In Go, you can create Condition Variable using `sync.Cond` type.

It has one contructor function(`sync.NewCond()`) which takes a `sync.Locker` (usually simple mutex). Also it has three simple methods namely `Wait()`, `Signal()` and `Broadcast()`.

Thats the complete interface to Condition Variable in Go.

Coming back to the original problem. How to make a goroutine wait for some event?

## Better way: Use `sync.Cond`

`Wait()` and `Signal()` methods are used to wait and signal the go-routine respectively

```
type Record struct {
	sync.Mutex
	data string

	cond *sync.Cond
}

func NewRecord() *Record {
	r := Record{}
	r.cond = sync.NewCond(&r)
	return &r
}

func main() {
	var wg sync.WaitGroup

	rec := NewRecord()
	wg.Add(1)
	go func(rec *Record) {
		defer wg.Done()
		rec.Lock()
		rec.cond.Wait()
		rec.Unlock()
		fmt.Println("Data: ", rec.data)
		return
	}(rec)

	time.Sleep(2 * time.Second)
	rec.Lock()
	rec.data = "gopher"
	rec.Unlock()

	rec.cond.Signal()

	wg.Wait() // wait till all goutine completes
}

```

Interesting thing to note here is, there is no need for infinte loop anymore. Now the waiting go-routine get suspended (means Go scheduler can execute some other go-routine)

> It may look like that the goroutine waiting(`rec.cond.Wait()`) is holding the lock whole time(`rec.Lock()`), but its not, Internally `cond.Wait()` unlocks it and it locks it again only when it wakes up by other go routine.

`Signal()` notifies only one of the goroutines(longest waiting gouroutine) that are waiting on the condition variable.

But wait! I’m a gopher and I know better way to signal one groutine from an another(`HINT: Channels`).

The same example can be written using channels as follows.

```
type Record struct {
	sync.Mutex
	data string
}

func main() {
	var wg sync.WaitGroup

	ch := make(chan struct{})

	rec := &Record{}
	wg.Add(1)
	go func(rec *Record) {
		defer wg.Done()
		<-ch
		fmt.Println("Data: ", rec.data)
		return
	}(rec)

	time.Sleep(2 * time.Second)
	rec.Lock()
	rec.data = "gopher"
	rec.Unlock()
	ch <- struct{}{}

	wg.Wait() // wait till all goutine completes
}

```

Nice. Now here comes the important question.

> If this problem can be solved using channels, why do we need condition variable in the first place?

Enters `Broadcast()` method of `sync.Cond`.

Lets look at following senario.

Say there are multiple goroutines waiting for some content in the buffer, how can we *keep* *notifing* all these goroutines whenever there is a new content in the buffer?.

Condition variable is a perfect fit here.

Following example try to explain this.

```
type Record struct {
	sync.Mutex

	buf  string
	cond *sync.Cond

	writers []io.Writer
}

func NewRecord(writers ...io.Writer) *Record {
	r := &Record{writers: writers}
	r.cond = sync.NewCond(r)
	return r
}

func (r *Record) Prompt() {
	for {

		fmt.Printf(":> ")
		var s string
		fmt.Scanf("%s", &s)

		r.Lock()
		r.buf = s
		r.Unlock()

		r.cond.Broadcast()
	}
}

func (r *Record) Start() error {
	f := func(w io.Writer) {
		for {
			r.Lock()
			r.cond.Wait()
			fmt.Fprintf(w, "%s\n", r.buf)
			r.Unlock()

		}
	}
	for i := range r.writers {
		go f(r.writers[i])
	}
	return nil
}

func main() {
	f, err := os.Create("cond.log")
	if err != nil {
		log.Fatal(err)
	}
	defer f.Close()
	r := NewRecord(f)
	r.Start()
	r.Prompt()
}

```

Above code keep prompting for new data from the user.

Here `NewRecord` can be called with multiple `io.Writers`(say log files, buffer, network connection etc..); each one waiting in separate go-routine on the condition variable `r.cond`. Each time there is a new data, all those waiting go-routines get notified via `r.cond.Broadcast()`

You can also think of handling the same behaviour using channel’s `close()` operation, so that all the waiting goroutine will receive zero value and proceed further. But what if you want to broadcast multiple times? (you cannot close the channel that is already closed, it panics if you do)

Summary:

- Go provides condition variable via `sync.Cond`
- Condition variables are useful if all you want is to signal other goroutines that some event has occured.
- Use `Signal()` to notify one of the goroutines that is waiting for the longest period(Go uses FIFO data structure internally to keep track waiting goroutines on a condition variable).
- Condition variables are perfect fit if you need to broadcast that some event has occured to all the goroutines waiting for that particular event.