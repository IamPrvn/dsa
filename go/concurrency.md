[TOC]



#### synchronisation in go

```go
package main
import (
	"fmt"
    "sync"
    "runtime"
)
var (
	counter int64
  	wg sync.WaitGroup
)

func main() {
  wg.Add(2)
  go incCounter(1)
  go incCounter(2)
  wg.Wait()
  fmt.Println("Final Counter " , counter)  
}

func incCounter(id int) {
  defer wg.Done()
  for i := 1; i<=2; i++ {
    val := counter
    // yields the thread and placed back in queue
    runtime.Gosched()
    val++
    counter = val
  }
}

// o/p :-  Final Counter: 2
```

- Each goroutine overwrites the work of the other. This happens when the goroutine swap is taking place. Each goroutine makes  its own copy of the counter variable and then is swapped out for the other goroutine. When the goroutine is given time to execute again, the value of   the counter variable has changed, but the goroutine doesn’t update its copy. Instead it continues to increment the copy it has and set  the value back to the counter variable, replacing the work the other goroutine performed.


- run `go build -race`  to see the race condition.
- here `counter` was a shared resource, let's see various ways of locking shared resources.

##### Atomic functions

- Atomic functions provide low-level locking mechanisms for synchronizing access to integers and pointers.

```go
func incCounter(id int) {
  defer wg.Done()
  for i := 1; i<=2; i++ {
    atomic.AddInt64(&counter, 1)
    // yields the thread and placed back in queue
    runtime.Gosched()
  }
}

// o/p :-  Final Counter: 4
```

- Another crucial _atomic functions_ apart from _AddInt64()_ are _LoadInt64_ and _StoreInt64_.

```go
var {
  wg sync.WaitGroup
  shutdown int64
}

func main() {
  wg.Add(2)
  
  go doWork("A")
  go doWork("B")
  
  time.Sleep(1* time.Second)
  fmt.Println("Shutting now")
  atomic.StoreInt64(&shutdown, 1)
  wg.Wait()
}

func doWork(name String) {
  defer wg.Done()
  for {
    fmt.Printf("Doing some work %s", name)
    time.Sleep(250 * time.Millisecond)
    if atomic.LoadInt64(&shutdown) == 1 {
      fmt.Printf("Shutting Down %s", name)
      break;
    }
  }
}
```



##### Mutexes

- hold and release lock in critical section

```go
func incCounter(id int) {
  defer wg.Done()
  for i := 1; i<=2; i++ {
    mutex.Lock()
    {
    	val := counter
    	// yields the thread and placed back in queue
    	runtime.Gosched()
    	val++
    	counter = val
    }
    mutex.Unlock()
  }
}
```



##### Channels

- When declaring a channel, the type of data that will be shared needs to be specified.

  ```go
  // Unbuffered channel of integers.
  unbuffered := make(chan int)
  // Buffered channel of strings.
  buffered := make(chan string, 10)

  // Send a string through the channel.
  buffered <- "Gopher"
  // Receive a string from the channel.
  value := <-buffered
  ```

###### Unuffered channels

- For unbuffered channel, the sender will block on the channel until the receiver receives the data from the channel, whilst the receiver will also block on the channel until sender sends data into the channel. `unbuffered channel := make(chan int)`

-  channel with no capacity to hold any value before it’s received.

- you see an example of two goroutines sharing a value using an unbuffered channel. In step 1 the two goroutines approach the channel, but neither have issued a send or receive yet. In step 2 the goroutine on the left sticks its hand into the channel, which simulates a send on the channel. At this point, that goroutine is locked in the channel until the exchange is complete.  In step 3 the goroutine on the right places its hand into the channel, which simulates a receive on the channel. That goroutine is now locked in the channel until the exchange is complete. In steps 4 and 5 the exchange is made and finally, in step 6, both goroutines are free to remove their hands, which simulates the release of the locks. They both can now go on their merry way.

  ![image](https://www.safaribooksonline.com/library/view/go-in-action/9781617291784/06fig06_alt.jpg)





- **Example 1**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int)

    go func(ch chan int) {
        fmt.Println("Func goroutine begins sending data")
        ch <- 1
        fmt.Println("Func goroutine ends sending data")
     }(ch)

    fmt.Println("Main goroutine sleeps 2 seconds")
    time.Sleep(time.Second * 2)

    fmt.Println("Main goroutine begins receiving data")
    d := <- ch
    fmt.Println("Main goroutine received data:", d)

    time.Sleep(time.Second)
}
```

The running result likes this:

```go
Main goroutine sleeps 2 seconds
Func goroutine begins sending data
Main goroutine begins receiving data
Main goroutine received data: 1
Func goroutine ends sending data
```


- **Example 2** 

_In the game of tennis, two players hit a ball back and forth to each other. The players are always in one of two states: either waiting to receive the ball, or sending the ball back to the opposing player._

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

var (
	wg sync.WaitGroup
)

func init() {
	rand.Seed(time.Now().UnixNano())
}

func main() {
	wg.Add(2)
	court := make(chan int)
	go player(court, "Nadal")
	go player(court, "Federar")
	fmt.Printf("Staring match\n")
	// both players will wait on each other, hence we are starting the match
  	court <- 1
	wg.Wait()
}

func player(c chan int, name string) {
	defer wg.Done()

	for {
		ball, ok := <-c
		if !ok {
			fmt.Printf("Player %s won\n", name)
			return
		}
		n := rand.Intn(100)
		if n%13 == 0 {
			fmt.Printf("Player %s missed\n", name)
			close(c)
			return
		}
		ball++
		fmt.Printf("Player %s hit the ball %d\n", name, ball)
		c <- ball
	}

}
```

- **Example 3**

  _In a relay race, four runners take turns running around the track. The second, third, and fourth runners can’t start running until they receive the baton from the previous runner. The exchange of the baton is a critical part of the race and requires synchronization to not miss a step. For this synchronization to take place, both runners who are involved in the exchange need to be ready at exactly the same time._



###### Buffered channels

- Compared with unbuffered counterpart, the sender of buffered channel will block when there is **no** empty slot of the `channel`, while the receiver will block on the channel when it is empty.

  `bufferedChannel := make(chan int, 5)`

  while receiving do a range over channels :

  `for data := range  bufferedChannel {}` 


- a channel with capacity to hold one or more values before they’re received
- These types of channels don’t force goroutines  to be ready at the same instant to perform sends and receives. 
- An unbuffered channel provides a guarantee that an **exchange between two goroutines is performed** **at the instant the send and  receive take place** (Note a goroutine also writes to its channel in its goroutine ). A buffered channel has no such guarantee.

```go

```




#### notes

- **co-operative vs pre-emptive**

  In a co-operative system a task will continue until it explicitly relinquishes control of the CPU. In a pre-emptive model tasks can be forcibly suspended. This is instigated by an interrupt on the CPU. These interrupts may be from external systems as above or possibly from the system clock.

  So you might have seen `runtime.Gosched()` yields. It means  that it asks the scheduler to kick in and run any one of the "ready" goroutines in an intentionally unspecified selection order.

