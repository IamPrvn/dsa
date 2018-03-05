#### notes

- **co-operative vs pre-emptive**

  In a co-operative system a task will continue until it explicitly relinquishes control of the CPU. In a pre-emptive model tasks can be forcibly suspended. This is instigated by an interrupt on the CPU. These interrupts may be from external systems as above or possibly from the system clock.

  So you might have seen `runtime.Gosched()` yields. It means  that it asks the scheduler to kick in and run any one of the "ready" goroutines in an intentionally unspecified selection order .

- ​

