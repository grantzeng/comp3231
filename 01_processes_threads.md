# (Questions)
```
1. Invariants about processes/threads true about all kernels

2. Different ways different OSes implement these abstractions

    - focus on OS161 (since I need this to implement fork/exec for redoing the first assignment)
```

# Processes/threads
A _process_ is the abstraction that provides the illusion to executing code of having its own private machine. This means: 
1. Providing it with a (private) address space (resource context + memory isolation) 
2. Providing what appears to be a CPU that it can run on. (execution context)

> The goal is to multiplex programs on shared hardware to maximise utilisation while preventing interference (especially with the kernel and other processes).

### (An aside about threads)
In OS161, the _unit of execution_ is called a thread, and a process is really just the _resource container_ shared by threads or alternatively, "a process has one or more threads". 
- The scheduler schedules _threads_ not _processes_ in OS161. ("kernel threads" rather than "user threads", i.e. there is no other layer of scheduling per process of the threads of that process)

In this terminology of _threads per process_ you can classify operating systems like this: 
||Single process| Multiple processes|
|-|-|-|
|Single thread | MSDOS, simple embeddded|Traditional Unix|
|Multiple threads|(OS161 (as distributed), maybe some RTOSs; generally rare)|Modern Unix (Linux), Windows|

but whether/how this process/thread distinction holds depends on the specific OS you're looking at. 

> The point is we need some kind of schedulable execution context, but what gets treated as thread state or process state will vary depending on OS design. 

## Lifecycle 

### State machine
A live unit of execution ("thread of execution") is either `BLOCKED`, `READY` or `RUNNING`: 

The canonical transitions are:
1. `RUNNING` $\rightarrow$ `BLOCKED` 
2. `RUNNING` $\rightarrow$ `READY`
3. `BLOCKED` $\rightarrow$ `READY` 
4. `READY` $\rightarrow$ `RUNNING` 

> TODO: what triggers the state transitions 

### Creating/destroying 
> TODO: what causes proceses to be created/destroyed


## Context switching



# Implementing processes/threads

### What should we put into a process control block/PCB? What should we put in the TCB? 

<!-- > NOTE: Depending _on_ how your OS in question implements processes and threads, some of this stuff might be tracked in the thread control block (it's a question of whether you want to encapsulate this info at the process or the thread level)

_Process management_: register, `pc`, `psw`, `sp`, process statue, priority, scheduling parametres, `pid`, `ppid`, process group, signals, time started, cpu time used, children's cpu time, time of next alarm etc. 

_Memory managemement_: pointer to `.text`, pointer to `.data`, pointer to `.stack` etc. 

_File management_: root, `cwd`, `fd`s, `uid`, `gid` etc.  -->

<!-- 

### Example: OS161 process interface: `kern/include/proc.h`
The data structure for tracking the process information
```c
struct proc {  
    char *p_name; 

    // Locking the data structure
    struct spinlock p_lock; 

    unsigned p_numthreads; 

    // Virtual memory (in OS161 memory is virtualised)
    struct addrspace *p_addrspace; 

    // Current working directory (also work with virtualised file system)
    struct vnode *p_cwd; 

    // More stuff
}
```
Operations you can do on/with processes. 
- It's not very interesting because OS161 has kernel threads (the scheduler actually schedules the execution contexts directly, instead of it being done at the process level)
- All that's really is a memory context
```c
// Called during system set up to allocate data strucutres
void proc_bootstrap(void);  

void proc_destroy(struct proc *proc); 
struct *proc_create_runprogram(const char *name); 

// OS161 at UNSW is implemented as multithreaded
int proc_addthread(struct proc *proc, struct thread *t); 
void proc_remthread(struct thread *t); 

// Getting and setting address space (probably useful to implement fork)
struct addrspace *proc_getas(void); 
struct addrspace *proc_setas(struct addrspace *); 
```

> TODO: For OS161, you'll ahve to look more into `kern/include/thread.h`

### Exanple: `xv6` processes

```c
enum procstate { UNUSED, USED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE }; 

struct proc { 
    // Data structure lock
    struct spinlock lock; 
    
    // Need to hold p->lock when modifying; 
    enum procstate state; 
    void *chan;             // if non-zero it's sleeping on some channel
    int killed;             // If non-zero process has been killed
    int xstate;             // exit status returned to parent's wait

    int pid; 

    // Need to hold wait_lock before modifying. 
    struct proc *parent; 

    // Items private to processes 
    uint64 kstack; 
    uint64 sx;                      // size of process memory in bytes
    pagetable_t pagetable; 
    struct trapframe *trapframe; 
    struct context context; 
    struct file *ofile[NOFILE];     // open file "table"
    struct inode *cwd; 
    char name[16]; 

}

```

System calls that affect current running processes

### Typical Unix interface for manipulating a process
```c
// Create + run programs: Unix model is "fork the current procesws, then exec" 
fork()              // create a new process
execve()            // replace current process image wiht a new program 
exit(status)        // terminate process

// Waiting for a child process and then exit
wait()
waitpid()

// Asynchronous control 
kill(pid, sig)      // send signal to a process
signal()            // install handlers
sigaction()
sigprocmask()       // block/unblock signals 

// Process identity/groupings
getpid()           
getppid()
setpgid()           // process group id
getpgid()      


// high level scheduling priority control 
nice()
setpriority()


// other things that change processes
chdir()
getwd()
getenv()            // environment needs to be passed to execve() (file descriptors need to be inherited to a child process so you have open/close/read/write, dup/dup2, fcntl etc.)


// IPC/synchronisation 
pipe() + fork()     // how shells work (pipe for IPC)
socketpair()        // shared memory, message queues etc.

``` -->


<!-- ### Implementing `fork`/`exec`/`wait`/`exit`, `kill` etc -->
<!-- 
# Threads

### Processes vs. threads -->
