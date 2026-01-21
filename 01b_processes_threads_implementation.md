# Implementing processes/threads 

The working mental model is this: 
- A _process_ bundles both execution/s and execution contexts (it is _not_ just a container)
- A _thread_ always belongs to some process, and refers to a specific path of sequential execution. 

## Hardware



## How/what to bookkeep?
> A lot of this comes down to: what do you want to track with the container and what do you want to track for paths of execution? 

### Process model
> Basically, in traditional Unix, there is one thread per process, and things to track about the container and the execution are bundled together. 

Under this model a _process control block_ will have things like: 
- _Process management metadata_:  registers, `pc`, `psw`, `sp`, process state, priority, scheduling parameters, `pid`, `ppid`, process group, signals, time start, cpu time used, children's cpu time, time of next alarm
- _Memory management_: pointers to `.text`, `.data`, `.stack` 
- _File management_: `root`, `cwd`, `fd`s ("open file table"), `uid`, `gid`

### Thread model  
Under the thread model, execution gets separated from the environment, so:  
- _Per process/"shared between threads"/"global"_: address space, global variables, open files, child processes, alarms, signals, signal handlers, other accounting information 
- _Per thread_: `pc`, registers, `sp`, thread specific state

> This separates container metadata from execution metadata. Sharing of the globals also now introduces some concurrency problems.


## Kernel/user threads





## High level implementation of the scheduler 
> Not discussing scheduling policy here
