# (How to use synchronisation primitives)
> some classic problems and a discussion of non-solutions and solutions

> Problem: trying to code under "adversarial" interleavings of code, and where there is some shared resource. 

> Unlike in say hardware, we can't necessarily rely on a clock to keep everything in sync. so you have to program around this. 

# Producer-consumer/bounded buffer problem 
There is some fixed sized buffer, there are producers that produce data items and store in the buffer, and consumers that take items out of the buffer. 

Ideally want this behaviour: 
- Producer (1) sleeps when buffer is full (2) wakes up when there is an empty space (e.g. some consumer calls wakeup when it consumes first entry of a full buffer)
- Consumer (1) sleeps when buffer is empty and (2) wakes up when items available (e.g. some producer calls wakeup when it adds a first item to buffer)
### A non-solution 

```c
// Ideally we want to write something like this, but there's problems 

#define N 4 // buffer size
int count = 0; 

void prod() {
    while (TRUE) {
        item = produce(); 
        if (count == N) 
            sleep(prod);
        insert_item(item);
        count++; 
        if (count == 1) 
            wakeup(con); 
    }
}

void con() {
    while(TRUE) {
        if (count == 0)   
            sleep(con); 
        remove_item(); 
        count--; 
        if (count == N-1) 
            wakeup(prod); 
    }
}

```
The obvious issue is that uncontrolled concurrent access to the buffer _and_ the counter will lead to data races

#### Lost wakeups
A more subtle issue is that of the **lost wakeup** (because of how the scheduler might randomly interleave code)
- Suppose count is `N` and `prod`. _As_ it tests for `count == N`, suppose it swaps to a consumer. 
    - The issue here is that _inspecting_ the metadata isn't coupled with the action that _updates_ it. Not atomic!
- The consumer _removes_ an item, and decrements count to `N-1` and then calls `wakeup(prod)` because it passes the test and then suppose the scheduler swaps back
- but then `prod` calls `sleep(prod)` and we've lost a wake up!

The situation is symmetric with `con` testing `count == 0` but before going to sleep, and `prod` running to prodce a wake up, and then swapping back to going to sleep. 

### Another non-solution 
`Issue with this "solution" you've just ended up with some other shared state`

### Solving it using semaphores
The invariant we have to maintain is that `items + space <= N` (total threads accessing buffer can't be greater than `N`)

```c
#define N 4 // buffer size

// Have a lock to protect the buffer from concurrent insert/remove
// - using a semaphore as a "lock"
struct semaphore *mutex = sem_create("", 1); 

// Have two semaphores to free and empty slots 
// - if there are free slots, let in producers
// - if there are empty slots, let in consumers
struct semaphore *items = sem_create("", 0); // how many many filled slots
struct semaphore *space = sem_create("", N); // how many free slots

void producer(void) { 
    while (True) { 
        item = produce(); 

        P(space); // Wait until there is space
    
        P(mutex); // Lock buffer to prevent concurrent access
        insert_item(item); 
        V(mutex);  

        V(items); // Notify there is an item
    }
}

void consumer(void) { 
    while (True) {
        P(items);  // Wait until there is an item   

        P(mutex); 
        remove_item(); 
        V(mutex); 

        V(space);  // Notify there is a space
    }
}

```


### Solving it with condition variables

`EXERCISE`
```c
#define N 4

struct lock *l = lock_create(""); // this can be used to lock both the buffer and the condition variable. The point is tying an action with updating the metadata 
struct cv *cv = cv_create("");  // single queue that queues up both producers and consumers (not nice)
int n = 0; 

void producer(void) {
    while (True) {
        item = produce_item(); 
        
        lock_acquire(l); 
        
        while (n == N) { // while full, wait
            cv_wait(cv, l); 
        }
        
        n++; 
        insert_item(item); 
        
        cv_broadcast(cv, l); // not ideal because wakes both consumers and producers ("herd of threads wake up for the resource")
        
        lock_release(l); 
    }
  
}

void consumer(void) {
    while(True) { 
        lock_acquire(l); 
        
        while (n == 0) {  // while empty, wait
            cv_wait(cv, l); 
        }
        
        n--; 
        remove_item(); 
        
        cv_broadcast(cv, l); 
        
        lock_release(l); 
    }
}

```
We can do it with one condition variable, but the trouble is this wakes both consumers and producers, ideally we would have separate queues for consumers and producers. 

Canonical version is to separate the two. 

```c
#define N 4 

struct lock *l = lock_create(""); // single lock to protect buffer and cvs
struct cv *items = cv_create("");  // queue up consumer threads
struct cv *spaces = cv_create("");  // queue up producer threads
int n = 0; 

void producer(void) { 
    while (True) {
        item = produce_item(); 
        
        lock_acquire(l); 
        
        while (n >= N) { 
            cv_wait(spaces, l); // go to sleep until there is an item
        }
        
        n++; 
        insert_item(item); 

        cv_signal(items, l); // notify there is an item
        
        lock_release(l); 
    }
}

void consumer(void) {
    while (True) { 
        lock_acquire(l); 

        while (n == 0) {  // we need this to deal with spurious wakeups
            cv_wait(items, l); s
        }

        n--; 
        remove_item(); 

        cv_signal(spaces, l); // notify there is a space
        
        lock_release(l); 
    }

}

```

# Readers-writers problem

# Dining philosophers
