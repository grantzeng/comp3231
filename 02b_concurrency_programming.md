# Handling concurrency
> 1. Discussion of synchronisation primitives and 2. how to actually use them. 

### (The pattern of resource counting/locking in design of data structures)



# Synchronisation primitives zoo
> Interface/header file lives at `kern/include/synch.h` 

These are the primitives we use instead because they allow threads to be put to sleep/waking, instead of having to spin/constantly poll for whether a resource is available. 


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

**Use it for permit counting.** Semaphore is initialised to the number of permits to access a resource. 

> If you set the initial count to 1 ("binary semaphore") then effectively you have a lock/mutex: at most one thread is allowed to access a resource.


## Mutexes/locks
```c
// The word lock is overloaded generally, but in OS161, lock ~= thing you use for mutexing
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
**Use it for mutual exclusion** on a resource. 

## Condition variables
```c
struct cv {
    char *cv_name; 
    struct wchan *cv_wchan; 
    struct spinlock cv_wchanlock; 
}

struct cv *cv_create(const char *name); 
void cv_destroy(struct cv *); 

void cv_wait(struct cv *cv, struct lock *lock); 
void cv_signal(struct cv *cv, struct lock *lock); 
void cv_broadcast(struct cv *cv, struct lock *lock); 
```

**Use it to make a thread wait for a predicate about some particular variable to become true**, instead of spinning until it does. (_It's not a varible itself_.)




## Monitors 
(Not available in OS161)

# How to use
> How some classic concurrency problems are solved

# Producer-consumer/bounded buffer problem 

# Readers-writers problem

# Dining philosophers

