# `fork`ing hell

> _Note_: see `https://www.microsoft.com/en-us/research/wp-content/uploads/2019/04/fork-hotos19.pdf` (Today, fork is bad)


> Unix: single threaded process with small memory footprint (start as this as your model organism for an operating system)

_Why do this?_ You have to touch a lot of different subsystems in the OS to get this to work and really understand how the process abstractions works. Also I want to get out of theory land and into build-stuff land. 


### Interface to implement 
```
fork
getpid
copy_splice
waitpid
_exit
kill_curthread
execv
```

This is to make the base implementation of OS161 less useless. 

### Invariants `fork` must maintain 

