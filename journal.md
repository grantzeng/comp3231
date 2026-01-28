# 2026-01-19 
### 11:45am 

Not really happy with my notes about threads vs. processes, I feel like I haven't really captured kernel of things I'd need to say about them.


### 12:08pm 

The three main conceptual areas to revise: 
1. Processes (container/execution context), threads (unit of execution)
2. Concurrency (esp. how to program it safely)
3. Memory management + virtual memory
4. Syscalls (crossing the kernel/user boundary)


# 2026-01-21 
### 11:39am 
Some difficulty of the correspondence between the container/execution mental model I have, and the process/thread discusssion. 



### 12:17pm 
I think take a _short break_. Then come back and jot down a short list of the key concepts I'd want to be able to explain for processes/threads. 


# 2026-01-26 
### 12:31pm 
Today I will focus on recapping the theory of the hardware that I need to know about to do OS work. 

### 2:19pm 
Note: separate out the topics to different files, but you should make a catalogue for notes and how they match to each week

### 2:25pm
We're going to stop here and take a break.

```
Aims for this week 
[x] minimum amount of computer architecture needed for OS
[] minimum amount of knowledge of the memory subsystems needed for OS 
[] basic discussion about how to implement the different abstractions
[] EOS: more advanced discussion about how to implement the abstractions (scheduler activations etc.)

Last week
[x] notes discussing the fact we need abstractions for multiplexing a CPU
```


# 2026-01-28 
### 2:17pm 
I'm going around in circles re: processes and threads implementation. Basically, the issue is this: somehow the CPU has to be multiplexed  - what are some reasonable ways of solving this. 

There are three standard solutions: 
- processes (isolated container with single thread of execution)
- kernel threads (we, the OS/kernel has to manage the concurrency within each container)
- user threasd (let the user deal with managing the concurrency)



I think just learn this + what are the trade-offs for each design
> _Why I am doing this_: For AOS, you're building an OS so you will probably need to implement some kind of abstraction of the CPU 


### 5:02pm 
Come back to implementation theory. 

Basically, at high level what interfaces + data structures have to be offered. 

The EOS lecture for processes and threads is just an extension of this. 
