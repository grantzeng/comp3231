# Concurrency

## Critical regions, race conditions

A **critical region** is a region of code where shared resources are accessed. 

The _issue_ with uncoordinated access is that you get **race conditions**, that is that outcome of the code is now non-deterministic/depends on what order the different threads interleave in execution. 

> _Classic example_ is a data race between two threads one trying to increment a global variable and one trying to decrement a global variable. It's not even the _interleaving_ of the lines of code, it's also of the assembly because the reading/writing is non-atomic!

# Synchronisation 

Design criteria for synchronising processes/threads
```
(1) _Mutual exclusion_: No two processes can be simulataneously inside their critical regions (mutual exclusion)
(2) Don't want to make any assumptions about speed/number of CPUs/treat scheduler as random.
(3) Processes running _outside_ their critical region should not block any other process
(4) _No starvation_:  No process should have to wait forever to enter critical region 
(5) (Implicitly: security)
(6) (Implicitly: don't waste resources)
```
## (Some non/non-ideal solutions) 
_Disabling interrupts_: Not acceptable to do at user level &mdash; security, cannot guarantee (3) and (4); possibly useful at kernel level but this does not work on a multiprocessor. 

_Lock variable_: The lock variable becomes a new bit of shared global state, so it even fails (1). The issue is that naively read/writing a variable is non-atomic. 

_Strict alternation_: The idea is processes busy wait/spinlock until their turn which fails (6). Process also gets a turn regardless of whether they're in the critical region or not so this fails (3). 

_Petersen's solution_: `(Omitted)`

> But as far as I can tell the issue is that it involves a busy wait = generating heat 

_Test-and-set lock_: Basically a lock variable, but use hardware instructions to lock the memory bus atomically. Fails (5) at user level, but useful in kernel. 
- The problem with this solution is basically it involves busy-waiting/constantly polling the lock variable until you see it's your turn. 
- Starvation is also possible if more than one process is waiting for the resource (it's whoever gets there first that wins)



## Locks 
??? 

## Semaphores 
Really what we want to introduce is the semantics of putting a thread to *sleep* if a resource is not available and *waking* it when it is (to avoid the spinning/busy wait)

```C
struct semaphore {
    char *sem_name; 
    struct wchan *sem_wchan;   // "queue" of block threads
    struct spinlock sem_lock;  // for protecting sempahore data structure
    volatile int sem_count;    // how many threads are "allowed in the room" 
};
 
struct semaphore *sem_create(const char *name, int initial_count);

void sem_destroy(struct semaphore *);

/*
    Semaphore operations (necessarily atomic)
    
    P: "proberen"/decrement/wait/down 
        - decrements until 0, and then blocks until count is 1 again
        - (if set this to 1, then you have a binary semphore to implement a mutex)

    V: "verhogen"/increment/signal/up
        - increments count
*/ 
void P(struct semaphore *); 

void V(struct semaphore *); 
```

_However_, programming with them is difficult because every signal has to matched to some wait (what if you have too many or miss some?), ordering of signals/waits can cause issues 

> Analogous to matching `malloc` with `free`

## Condition variables 
Ideally we may want to wait until some predicate is true. 

> ? Solves issue of lost wakeup? 
> I never used condition variables, so we need to do a little practice with this

```c
struct cv {
    char *cv_name;
    struct wchan *cv_wchan;
};

struct cv *cv_create(const char *name);

void cv_destroy(struct cv *);

/*
    Condition variable operations (must be atomic) 

    - cv_wait - release supplied lock, sleep; and if woken, reacquire lock
    
    - cv_signal - wake up one thread that's sleeping 
    
    - cv_broadcast - wake up ALL threads that're sleeping 
*/
void cv_wait(struct cv *cv, struct lock *lock);

void cv_signal(struct cv *cv, struct lock *lock);

void cv_broadcast(struct cv *cv, struct lock *lock);
```

## Monitors

# Classic problems

## Producer-consumer bounded buffer problem


## Dining philosopher's problem 








# Deadlock 
# Livelock 


