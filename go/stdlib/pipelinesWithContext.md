[TOC]



#### working with context package



- As we’ve seen, in concurrent programs it’s often necessary to preempt operations because of timeouts, cancellation, or failure of another portion of the system. We’ve looked at the idiom of creating a done channel, which flows through your program and cancels all blocking concurrent operations. This works well, but it’s also somewhat limited.

- Following is the skeleton of a context package :-

  ```go
  var Canceled = errors.New("context canceled")
  var DeadlineExceeded error = deadlineExceededError{}

  type CancelFunc
  type Context

  func Background() Context
  func TODO() Context

  func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
  func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
  func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
  func WithValue(parent Context, key, val interface{}) Contexttype Context
  ```

  ##### type Context

  ```go
  type Context interface {    
    // Deadline returns the time when work done on behalf of this    
    // context should be canceled. Deadline returns ok==false when no    
    // deadline is set. Successive calls to Deadline return the same    
    // results.    
    Deadline() (deadline time.Time, ok bool)    
    
    // Done returns a channel that's closed when work done on behalf    
    // of this context should be canceled. Done may return nil if this    
    // context can never be canceled. Successive calls to Done return    
    // the same value.
    Done() <-chan struct{}    
    
    // Err returns a non-nil error value after Done is closed. Err    
    // returns Canceled if the context was canceled or    
    // DeadlineExceeded if the context's deadline passed. No other    
    // values for Err are defined.  After Done is closed, successive    
    // calls to Err return the same value.
    Err() error    
    
    // Value returns the value associated with this context for key,
    // or nil if no value is associated with key. Successive calls to
    // Value with the same key returns the same result.
    Value(key interface{}) interface{}}
  ```

  -  There’s a Done method which returns a channel that’s closed when our function is to be preempted.
  - a Deadline function to indicate if a goroutine will be canceled after a certain time, and an Err method that will return non-nil if the goroutine was canceled
  - Usually in these programs, request-specific information needs to be passed along in addition to information about preemption. This is the purpose of the Value function.

- As we mentioned, the Context type will be the first argument to your function. If you look at the methods on the Context interface, you’ll see that there’s nothing present that can mutate the state of the underlying structure. Further, there’s nothing that allows the function accepting the Context to cancel it. This protects functions up the call stack from children canceling the context. Combined with the Done method, which provides a done channel, this allows the Context type to safely manage cancellation from its antecedents.

  #### methods in context package

  ##### WithCancel

  ```go
  func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
  ```

  WithCancel returns a new Context that closes its done channel when the returned cancel function is called

  ##### WithDeadline

  ```go
  func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
  ```

  `WithDeadline` returns a new `Context` that closes its `done` channel when the machine’s clock advances past the given `deadline`

  ##### withTimeout

  ```
  func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
  ```

  WithTimeout returns a new Context that closes its done channel after the given timeout duration.

- If your function needs to cancel functions below it in the call-graph in some manner, it will call one of these functions and pass in the Context it was given, and then pass the Context returned into its children. If your function doesn’t need to modify the cancellation behavior, the function simply passes on the Context it was given.

  ​

  ##### Background & TODO

  At the top of your asynchronous call-graph, your code probably won’t have been passed a Context. To start the chain, the context package provides you with two functions to create empty instances of Context.

  ```go
  func Background() Context
  func TODO() Context
  ```

  Background simply returns an empty Context. TODO is not meant for use in production, but also returns an empty Context; TODO’s intended purpose is to serve as a placeholder for when you don’t know which Context to utilize, or if you expect your code to be provided with a Context, but the upstream code hasn’t yet furnished one.



#### Illustration :-

##### with done channel

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup
	done := make(chan interface{})
	defer close(done)

	wg.Add(1)
	go func() {
		defer wg.Done()
		if err := printGreeting(done); err != nil {
			fmt.Printf("%v", err)
			return
		}
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		if err := printFarewell(done); err != nil {
			fmt.Printf("%v", err)
			return
		}
	}()

	wg.Wait()
}

func printGreeting(done <-chan interface{}) error {
	greeting, err := genGreeting(done)
	if err != nil {
		return err
	}
	fmt.Printf("%s world!\n", greeting)
	return nil
}

func printFarewell(done <-chan interface{}) error {
	farewell, err := genFarewell(done)
	if err != nil {
		return err
	}
	fmt.Printf("%s world!\n", farewell)
	return nil
}

func genGreeting(done <-chan interface{}) (string, error) {
	switch locale, err := locale(done); {
	case err != nil:
		return "", err
	case locale == "EN/US":
		return "hello", nil
	}
	return "", fmt.Errorf("unsupported locale")
}

func genFarewell(done <-chan interface{}) (string, error) {
	switch locale, err := locale(done); {
	case err != nil:
		return "", err
	case locale == "EN/US":
		return "goodbye", nil
	}
	return "", fmt.Errorf("unsupported locale")
}

func locale(done <-chan interface{}) (string, error) {
	select {
	case <-done:
		return "", fmt.Errorf("canceled")
	case <-time.After(1 * time.Minute):
	}
	return "EN/US", nil
}

```

- Running this code produces:

  goodbye world!

  hello world!

- Ignoring the race condition (we could receive our farewell before we’re greeted!), we can see that we have two branches of our program running concurrently. We’ve set up the standard preemption method by creating a done channel and passing it down through our call-graph. If we close the done channel at any point in main, both branches will be canceled.

- By introducing goroutines in main, we’ve opened up the possibility of controlling this program in a few different and interesting ways. Maybe we want genGreeting to time out if it takes too long. Maybe we don’t want genFarewell to invoke locale if we know its parent is going to be canceled soon. At each stack-frame, a function can affect the entirety of the call stack below it.

##### with context package

Let’s say that genGreeting only wants to wait one second before abandoning the call to locale—a timeout of one second. We also want to build some smart logic into main. If printGreeting is unsuccessful, we also want to cancel our call to printFarewell. After all, it wouldn’t make sense to say goodbye if we don’t say hello!

```go
package main

import (
	"context"
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup
	ctx, cancel := context.WithCancel(context.Background()) // <1>
	defer cancel()

	wg.Add(1)
	go func() {
		defer wg.Done()

		if err := printGreeting(ctx); err != nil {
			fmt.Printf("cannot print greeting: %v\n", err)
			cancel() // <2>
		}
	}()

	wg.Add(1)
	go func() {
		defer wg.Done()
		if err := printFarewell(ctx); err != nil {
			fmt.Printf("cannot print farewell: %v\n", err)
		}
	}()

	wg.Wait()
}

func printGreeting(ctx context.Context) error {
	greeting, err := genGreeting(ctx)
	if err != nil {
		return err
	}
	fmt.Printf("%s world!\n", greeting)
	return nil
}

func printFarewell(ctx context.Context) error {
	farewell, err := genFarewell(ctx)
	if err != nil {
		return err
	}
	fmt.Printf("%s world!\n", farewell)
	return nil
}

func genGreeting(ctx context.Context) (string, error) {
	ctx, cancel := context.WithTimeout(ctx, 1*time.Second) // <3>
	defer cancel()

	switch locale, err := locale(ctx); {
	case err != nil:
		return "", err
	case locale == "EN/US":
		return "hello", nil
	}
	return "", fmt.Errorf("unsupported locale")
}

func genFarewell(ctx context.Context) (string, error) {
	switch locale, err := locale(ctx); {
	case err != nil:
		return "", err
	case locale == "EN/US":
		return "goodbye", nil
	}
	return "", fmt.Errorf("unsupported locale")
}

func locale(ctx context.Context) (string, error) {
	select {
	case <-ctx.Done():
		return "", ctx.Err() // <4>
	case <-time.After(1 * time.Minute):
	}
	return "EN/US", nil
}
```

`1`	Here main creates a new Context with context.Background() and wraps it with context.WithCancel to allow for cancellations.

`2`  On this line, main will cancel the Context if there is an error returned from printGreeting.

`3`  Here genGreeting wraps its Context with context.WithTimeout. This will automatically cancel the returned Context after 1 second, thereby canceling any children it passes the Context into, namely locale.

`4`  This line returns the reason why the Context was canceled. This error will bubble all the way up to main, which will cause the cancellation at `2` 

Here are the results of running this code:

```
cannot print greeting: context deadline exceeded
cannot print farewell: context canceled
```

 Let us the locale defination to include deadline too :-

```go
func locale(ctx context.Context) (string, error) {
  if deadline, ok := ctx.Deadline() ; ok {
    if deadline.Sub(time.Now().Add(1*time.Minute)) <= 0 {
      return "", "context deadline exceeded"
    }
    select {
	case <-ctx.Done():
		return "", ctx.Err() // <4>
	case <-time.After(1 * time.Minute):
	}
	return "EN/US", nil
   }
}
```

