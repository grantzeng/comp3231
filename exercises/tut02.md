## Tutorial Week 3

### Synchronisation Problems

The following problems are designed to familiarise you with some of the problems that arise in concurrent programming and help you learn to identify and solve them.

---

# Coordinating activities

1. What synchronisation mechanism or approach might one take to have one thread wait for another thread to update some state?

Using semaphores
```c
// This is a wrong answer: because it's set to 1, t1 won't wait because there is a free slot and t1 will just run
// struct semaphore *sem = sem_create("", 1); 

// This is the right answer: correct because no permits initially
struct semaphore *sem = sem_create("", 0); 

void t0() {
    // do t0 stuff 
    V(sem); // signal t0 is done/increment number of permits
}

void t1() {
    P(sem);  // t1  blocked until semaphore count is non-zero/there is a permit 
    // safe to do t1 stuff
}
```
Using condition variables
```c
struct lock *l = lock_create(""); 
struct cv *cv = cv_create(""); 
int flag = 0; // don't go

void t0(){
    lock_acquire(l); 
    
    flag = 1; 
    cv_signal(cv, l); 

    lock_release(l); 
}

void t1() { 
    lock_aquire(l);
    
    while (flag != 1) {
        cv_wait(cv, l); 
    }

    // 

    lock_release(l); 
}

```

2. A particular abstraction only allows a maximum of 10 threads to enter the "room" at any point in time. Further threads attempting to enter the room have to wait at the door for another thread to exit the room. How could one implement a synchronisation approach to enforce the above restriction?

Using semaphores

```c
struct semaphore *sem = create_sem("", 10); 

void t() {
    P(sem);  // wait/decrement/proberen
    // - if it can decrement, will enter room 
    // - if decrementing would make count negative, put current thread to sleep

    // do thread stuff 

    V(sem);  // signal/increment/verhogen
}
```
```c
struct semaphore *room_sem = create_sem("", 10); 

// User has to make sure 
void enter() {
    P(sem); 
}

void leave() {
    V(sem); 
}

```



Using condition variables
```c
struct lock *l = lock_create(""); 
struct cv *cv = cv_create(""); 
int n = 0;  // threads in room

void t() {

    // enter room 
    lock_acquire(l); 
    while (n > 10) {  // wait if room is full
        cv_wait(cv, l); 
    }
    n++; 
    lock_release(l); 

    /*
    
        Do thread stuff
    
    */

    // leave room 
    lock_acquire(l);
    KASSERT(n > 0);  
    n--; 
    cv_signal(cv, l);  
    lock_release(l); 
}
```

```c
#define ROOM_CAPACITY 10 
struct lock *l = lock_create(""); 
struct cv *cv = cv_create(""); 
int n = 0; // invariants: 0 <= n <= 10

void enter() {
    lock_acquire(l); 

    while (n >= ROOM_CAPACITY) {
        cv_wait(cv, l); 
    }

    n++; // enter room
    
    lock_release(l); 
}

void leave() {
 
    lock_acquire(l);

    KASSERT(n > 0);  // I think this is nicer than using an if statement, the issue is int isn't tight enough a type to enforce domain of room count

    n--;  // leave the room
    cv_signal(cv, l); 

    lock_release(l); 
}

```


3. Multiple threads are waiting for the same thing to happen (e.g. a disk block to arrive from disk). Write pseudo-code for a synchronising and waking the multiple threads waiting for the same event.

> The point is to show when semaphores and condition variables are easy or hard. This one is hard with semaphores, because you're basically having to implement `cv_broadcast` from scratch. 

### Semaphore solution 
#### Seemingly correct but non-solution
```c
/*
    Strategy: 
    - need a semaphore to guard the disk 
    - need a semaphore to guard the count of waiting threads

*/

struct semaphore *disk_sem = sem_create("", 0);

int n = 0; 
struct semaphore *n_lock = sem_create("", 1); 

void wait() { /* enqueue a thread waiting for a disk*/

    P(n_lock);      // mark as waiting
        n++;  
    V(n_lock);

    P(disk_sem);    // enqueue for the disk 
}

void release() { /* called by disk when block arrives */
    
    P(n_lock);

    while (n > 0) { 
        V(disk_sem); 
        n--; 
    }

    V(n_lock); 
}
```
It's not at all obvious to me that this is wrong: 
- Might be a lost wake up 
- What if `release` is called before a `wait` (i.e. user doesn't match things)

> One problem: in `wait` there is a timing gap between incrementing the waiting count and actually waiting, so it could be the case the scheduler switches to to `release` and does a `V(disk_sem)`, before swapping back to `wait` and running `P(disk_sem)` $\rightarrow$ lost wake up 


#### Trying again 


```c
#define READY 1
#define NOT_READY 0

struct semaphore *mutex = sem_create("", 1); // metadata lock 
int status = NOT_READY; 
int n = 0; // count of waiters

struct semaphore *waiting = sem_create("", 0); // resource counting

void wait(void) {
    P(mutex); 
    
    if (status == NOT_READY) {  // we need to recheck status on wakeup
        n++; 
        V(mutex);  // release the metadata lock (so you don't go to sleep holding the mutex)
        P(waiting);  // take a permit and go to sleep 
        return; 
    }
    
    V(mutex); 
    
}

void release(void) {

    P(mutex); 
    
    status = READY; 

    while (n > 0) { 
        n--; 
        V(waiting); // release a permit and wake up someone up  
    }

    V(mutex); 

}
```

This solution solves the issue that updating the metadata on waiting wasn't coupled with actually waiting. 
- Mutex protects metadata/action coupling. 
- Then we just use `waiting` semaphore as a queue and  only need to issue `n` permits to release threads

### What happens if we allow block to be made unavailable 


```c
#define READY 1
#define NOT_READY 0

struct semaphore *mutex = sem_create("", 1); // metadata lock 
int status = NOT_READY; 
int n = 0; // count of waiters

struct semaphore *waiting = sem_create("", 0); // resource counting

void wait(void) {
    P(mutex); 
    
    if (status == NOT_READY) {  // we need to recheck status on wakeup
        n++; 
        V(mutex);  // release the metadata lock (so you don't go to sleep holding the mutex)
        P(waiting);  // take a permit and go to sleep 
        return; 
    }
    
    V(mutex); 
    
}

void release(void) {

    P(mutex); 
    
    status = READY; 

    while (n > 0) { 
        n--; 
        V(waiting); // release a permit and wake up someone up  
    }

    V(mutex); 

}

void prohibit(void) { 
    P(mutex); 
    status = NOT_READY; 
    V(mutex); 
}
```



### Condition variable solution 
Condition variable solution (condition on the status of the block)
```c
#define READY 1
#define NOT_READY 0

struct lock *l = create_lock(""); 
struct cv *cv = create_cv(""); 
int status = NOT_READY;  

void wait() { 
    lock_acquire(l);
    while (status == NOT_READY) {
        cv_wait(cv, l); 
    }
    lock_release(l); 
}

void release() {
    lock_acquire(l); 
    status = READY; 
    cv_broadcast(cv, l);  // notify all subscribed threads
    lock_release(l); 
}

```