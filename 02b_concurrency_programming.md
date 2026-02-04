# Handling concurrency
> 1. Discussion of synchronisation primitives and 2. how to actually use them. 

### (The pattern of resource counting/locking in design of data structures)



# Synchronization primitives
> Interface/header file lives at `kern/include/synch.h` 


> _What this is about_: Programming in contexts where there are multiple threads of execution (the issue is "bad" interleavings of code when there is some shared resource, because you should assume scheduler is random)

## Semphores 
> "Semaphores are the `goto` of concurrent programming"

> "Semaphores are still useful in other kinds of asynchronous processes such as CPU to GPU synchronization or keeping trains on railways from colliding." 

```c
struct semaphore { 
    char *sem_name; 
    struct wchan *sem_wchan; 
    struct spinlock sem_lock; 
    volatile unsigned sem_count; 
}

struct semaphore *sem_create(const char *name, unsigned initial_count); 

void sem_destroy(struct semaphore *);

void P(struct semaphore *);  // P: "proberen"/decrement/wait/acquire
void V(struct semaphore *);  // V: "verhogen"/increment/signal/release
```

There are really only two (questionable) use cases: 
1. **As a mutex/lock, when set as a binary semaphore**. If you set initial count to `1` then at most one resource is allowed in, so you can do mutual exclusion 
- Strictly speaking this isn't a lock, because locks are "owned". 

2. **For resource access counting/degree of concurrent access allowed**. Set initial count to how many threads you want to let access a resource initially. It might be `0` if you were, say protecting access to a resource not yet available. 

One of the picky issues is you need to make sure every `P` and `V` are matched otherwise you're in for trouble (analogous to `malloc/free` usage). 


It's hard to use when you're trying to make threads _wait_ on a particular condition. 
- Try implementing `cv_broadcast` with semaphores and you'll see (see tut02; basically you end up needing a mutex to protect the system metadata while it could be updated, then you need counting semaphore to track waiters)


## Mutexes/locks
```c
// The word lock is overloaded generally, but in OS161, lock ~= roughly thing you use for mutexing, but also has an owner
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

**When you're checking condition, you _must_ use a `while` loop**:  
```c
    lock_acquire(l); 

    // We actually want an if...but the issue is spurious wakeups! 
    // So need to recheck condition in case of that. 
    while (flag != 1) {
        cv_wait(cv, l)
    };

    lock_release(l); 
```


`cv_wait` works by releasing the lock (so someone else can inspect/modify the shared state), enqueueing current thread and then attempting to reacquire the lock 

> TODO: Look into _why_ it behaves like this, it's not very clear to me immediately why this is the case


## Monitors 
`(Not available in OS161)`

`TODO: FILL OUT THIS SECTION`


## Message passing/IPC
`(Omit as bonus content, was not discussed in course)`

