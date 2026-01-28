# Implementing different models for virtualising the CPU

> _What the original problem was:_ how do we make a computer look like it's running multiple programs at the same time (_virtualise the CPU_/_multiprogramming_), but also stop them from stomping all over each other (_resource isolation/management_)? 
> 
> There's more than one way to skin this cat, but it's impossible to talk about context switching without picking a _particular_ model. 
> 
> For the purposes of keeping scope manageable, we'll focus on how the process model, kernel threading and user threading are implemented, but the point is there are other ways of solving this problem
> 
> TODO: I'm also finding it hard to actually argue which model is better when. 
> 
> Honestly I think the discussion of threads complicates the matter, I would've left as something to discuss after discussing processes as an obvious extension. 


# Implementing the process model 
# Implementing the thread model 
## Implementing kernel threads
## Implementing user threads

# Context switching




<!--

Things to read up on : 
- minimum knowledge needed of hardware to get things working
- typical implementation strategies
    - kernel vs. user thread discussion 
- how context switching is implemented 
-->

<!-- > It's difficult to talk about implementation in general terms so we'll talk about how implementation would happen on MIPS R3000

> TODO: Look at how x86 works instead

## How does hardware affect the implementation? 
> See notes on MIPS R300. The only really important point is that this architecture branch delay slots (this is just a quirk specific to the hardware being emulated, not generally, e.g. x86 doesn't work like this) 

> TODO: I'm not _completely_ sure how this affects implementation, we'll get to that

# Abstractions for processes/threads

## Processes 

## Kernel threads

## User threads


# Context switching 

> This is hard to talk about in abstract, so we will go through specifically what OS161 does, which implements kernel threads. 



### When does it happen?

### How does it work?  -->
<!-- 
High level of what happens 
```
// What happens control gets passed to OS

1. Hardware saves pc, stack etc. // save where we are when interrupt occurs

2. Hardware loads new pc from interrupt vector  // load position for where execution to start for the OS

3. Assembly language procedure saves register // preserve state of the application before control is passed to OS so we don't corrupt it

4. Assembly language procedure sets up new stack // OS has its own stack so application can't corrupt the OS so this is all typically all coded in assembler

5. C interrupt service runs (read + buffer input)

6. Scheduler decides which process to run next

7. C procedure returns to assembly code 

8. Assembly language procedure starts up new current process
```

Context switching _threads_ is saving/restoring state associated with a thread. 

Context switching _processes_ is as above, but also extra state associated with a process. e.g.  memory maps -->


