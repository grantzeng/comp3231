# Handling concurrency
> 1. Discussion of synchronisation primitives and 2. how to actually use them. 

### (The pattern of resource counting/locking in design of data structures)



# Synchronization primitives
> Interface/header file lives at `kern/include/synch.h` 


> _What this is about_: Programming in contexts where there are multiple threads of execution 

## Semphores 
```c
struct sempahore { 
    char *sem_name; 
    struct wchan *sem_wchan; 
    struct spinlock sem_lock; 
    volatile unsigned sem_count; 
}

struct semaphore *sem_create(const char *name, unsigned initial_count); 

void sem_destroy(struct semaphore *);

void P(struct semaphore *);  // P: "proberen"/decrement/wait
void V(struct semaphore *);  // V: "verhogen"/increment/signal
```

**Use it for permit counting/"number of threads which can access a resource initially"** Semaphore is initialised to the number of permits to access a resource _initially_. 

> If you set the initial count to 1 ("binary semaphore") then effectively you have a lock/mutex: at most one thread is allowed to access a resource. But the difference is that locks are "owned", whereas a sempahore is not. 

One of the picky issues is you need to make sure every `P` and `V` are matched otherwise you're in for trouble (analogous to `malloc/free` usage). 


## Mutexes/locks
```c
// The word lock is overloaded generally, but in OS161, lock ~= roughly thing you use for mutexing
struct lock { 
    char *lk_name; 
    HANGMAN_LOCKABLE(lk_hangman); 
    struct wchan *lk_wchan; 
    struct spinlock lk_lock; 
    struct thread *volatile lk_holder; 
}
struct lock *lock_create(const char *name); 
void lock_destroy(struct lock *); 

void lock_acquire(struct lock *); 
void lock_release(struct lock *); 
bool lock_do_i_hold(struct lock *); 
```
**Use to mutually exclude threads** from a resource. 

## Condition variables
```c
struct cv {
    char *cv_name; 
    struct wchan *cv_wchan; 
    struct spinlock cv_wchanlock; 
}

struct cv *cv_create(const char *name); 
void cv_destroy(struct cv *); 

// Put current thread on wait queue
void cv_wait(struct cv *cv, struct lock *lock);

// Wake up one or all threads from the wait queue
void cv_signal(struct cv *cv, struct lock *lock); 
void cv_broadcast(struct cv *cv, struct lock *lock); 
```

**Use it to make a thread wait on a predicate to be true for some shared state**. It's badly named, we're _conditioning_ on some variable. The `cv` itself is basically a nicer wait queue.  

The _lock_ is to mutex accessing the shared state of the variable being watched. (You need to acquire/release)

When you're checking condition, you _must_ use a `while` loop:  
```c
    lock_acquire(l); 

    // We actually want an if...but the issue is spurious wakeups! 
    // So need to recheck condition in case of that. 
    while (flag != 1) {
        cv_wait(cv, l);
    }

    lock_release(l); 
```


`cv_wait` works by releasing the lock (so someone else can inspect/modify the shared state), enqueueing current thread and then attempting to reacquire the lock 

> TODO: Look into _why_ it behaves like this, it's not very clear to me immediately why this is the case


## Monitors 
`(Not available in OS161)`

`TODO: FILL OUT THIS SECTION`


## Message passing/IPC
`(Omit as bonus content, was not discussed in course)`

