# (Multiplexing the processor)
The problem we are trying to deal with is: how do you multiplex the processor? How do you "execute" multiple programs simultaneously. 

All these abstractions need some discussion of
- Sequential execution/s of a program ("thread of execution")
- Some kind of resource isolation/context in which the execution happens. ("container) 
with varying degrees of separation between the two concepts

The problem of _who/what_ to run is left to the scheduler. 

# Processes

The simplest way of doing this is to bundle an executing program with its context as a _process_. 

The idea is that the OS should provide an executing program the illusion that they have a private CPU to themselves. 

> The traditional Unix model: there are multiple processes, with only one thread of execution, so the distinction between a thread (execution) and a process (execution, plus its context/container) elided. 

The model is that while in implementation we have multiprogramming with one program counter, the conceptual model provided is that there are multiple isolated sequential processes each with its own program counter. 
- And at least on a uniprocessor, only one process is active at any moment. 


### Process lifecycle

> NOTE: At a high level this applies generally to any schedulable unit of execution (i.e. to threads too)

A live process is either `BLOCKED`, `READY` or `RUNNING`, and transition between these states like this: 
1. `RUNNING` $\rightarrow$ `BLOCKED`: waiting for I/O, a timer, or for some resource
2. `RUNNING` $\rightarrow$ `READY`: voluntary `yield`, scheduler picks a different process to run.
3. `BLOCKED` $\rightarrow$ `READY`: resource becomes available or I/O arrives. 
4. `READY` $\rightarrow$ `RUNNING`: scheduler picks this process to run. 

Processes are created during system initialisation (foreground (interactive processes), background (daemons)), when some other process executes a process creation syscall, when a user requests it or when a batch job is initiated
> In traditional Unix the kernel starts a special PID 0 idle/swapper task, then creates PID 1 (init). From there, almost all user processes are descendants of init (PID 1).

Processes termination is either _voluntary_ (normal exit, error exit) or  _involuntary_ (fatal error, killed by another process). 

### Why? 
> _Overall design motivation_: This prioritises memory isolation/protection boundaries which 
multiprogramming in the 1960s did not really have managed automatically. 

# Threads

Under the thread model, execution is distinguished from its context; a process becomes a resource container and we allow _some_ concurrency between the exeuction within the container.
- This includes every thread getting its own private stack (implicitly manage computation state)

In the terminology of _processes having threads_ we can classify operating systems thus: 
||Single process| Multiple processes|
|-|-|-|
|Single thread | MSDOS, simple embeddded|Traditional Unix|
|Multiple threads|(base OS161. but otherwise rare)|Modern Unix (Linux), Windows|

The reasons for wanting threading are:  
- _Performance_: Threads waiting for I/O can be overlapped with computing threads, which you can't if you only have one thread. 
> However there's no advantage if threads are compute bound on a uniprocessor, the point is we reduce idling the processor when waiting for I/O)
- If we have a multiprocessor, it's easier to take advantage of the parallelism underneath
- Threads are less resource intensive than whole processes, and can share resources/memory between them 
> But the trade-off to this is that you need to manage concurrent access to shared resources!
- (_Versus an event-driven system_) Simpler to program than a state machine. 


### Why? 
> _Overall design motivation_: want to allow some concurrency and shared memory (but less expensively than IPC). 
> - This might turn up in say, word processing software, where it makes sense to have conceptually separate threads e.g. one watching the keyboard, one watching the mouse, a daemon that saves the document etc.
> - Or a webserver, you have one infinite listening loop in a dispatcher thread that spawns worker threads when something needs to be done.  

# Event-driven systems 

The idea with this is that you multiplex the CPU across _many_ waiting computations

The downside is that the concurrency needs some explicit state manager, rather than in the individual stacks in threads where it is done implicitly.


### Why? 
> _Overall design motivation_: the workload involves massive amounts of I/O concurrency
> - I.e. most of the time you spend waiting for I/O, so there's not really any advantage to having all the additional resources to set up even a thread since it's not mostly about compute. 


<!-- # (Questions)
```
Scope: 
- True facts about processes/threads abstraction that are true of all kernels

See processes/threads implementation: this is OS161 specific details

Need to make notes on what is 3231 specific and what is generally true
```

### (Aside: terminology)

# Process

A _process_ is the abstraction that provides the illusion to executing code of having its own private machine. This means: 
1. Providing it with a (private) address space (resource context + memory isolation) 
2. Providing what appears to be a CPU that it can run on. (execution context)

This is because we want to multiplex programs on shared hardware to maximise utilisation, while preventing executing programs from interfering with each other or especially the kernel. 

### (Aside: threads)
> "A process has one or more threads"

The usual way of speaking about things is the _unit of execution_ is called a thread, and a process is really just the _resource container/execution context_ shared by threads. 

> Historically traditional Unix had singled threaded processes, so the distinction between execution ("thread") and execution/s with environment ("process") is often elided, and people think of a process as being execution as well, not just environment.  

In this terminology of _threads per process_ you can classify operating systems like this: 
||Single process| Multiple processes|
|-|-|-|
|Single thread | MSDOS, simple embeddded|Traditional Unix|
|Multiple threads|(rare, uncommon)|Modern Unix (Linux), Windows|

but how the process/thread distinction holds depends on the specific OS you're looking at. 

## Lifecycle 

### State machine 
A live scheduleable entity (thread or process depending on your OS) is either `BLOCKED`, `READY` or `RUNNING`

The canonical transitions are:
1. `RUNNING` $\rightarrow$ `BLOCKED`: waiting for I/O, a timer, or generally waiting for some resource
2. `RUNNING` $\rightarrow$ `READY`: voluntary `yield`, time slice finishes and scheduler picks something else to run
3. `BLOCKED` $\rightarrow$ `READY`: resource or I/O becomes available 
4. `READY` $\rightarrow$ `RUNNING`: scheduler picks this schedulable entity to run


### Creating/terminating
> TODO: Rewrite some of this is Unix specific

Processes are created during system initialisation (foreground (interactive processes), background (daemons)), when some other process executes a process creation syscall, when a user requests it or when a batch job is initiated
> In traditional Unix the kernel starts a special PID 0 idle/swapper task, then creates PID 1 (init). From there, almost all user processes are descendants of init (PID 1).

Processes termination is either _voluntary_ (normal exit, error exit) or  _involuntary_ (fatal error, killed by another process). 


### "Context switching"
A context switch happens when either threads are switched between or processes are switched between. 

> In OS161, the scheduler schedules _kernel threads_. This means only _sometimes_ does the container/parent process also need switching, since threads may both belong to one process. 

This happens any time: a system call is invoked, there is some exception, or on an interrupt. 

# Threads

 -->
