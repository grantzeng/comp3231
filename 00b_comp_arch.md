# (The absolute minimum background you need to work with OS code)

> Knowing about the ISA/minimum bits of computer architecture you need to know about to


> The point is it's kinda hard to talk generally about implementation without a specific piece of hardware, so we cover MIPS R3000. 


### MIPS R3000

Has a load/store architecture, only load and store instructions will ever operate on memory. 

### Load/store
You will mostly see `sw` and `lw`: load/store to/from memory from/to registers (but also `sh`/`lh`, `sb`/`lb` etc. for different sizes)
> A word is 32-bits. 


### Operations
There are no arithmetic/logic operations on memory; operations are only done to  registers e.g. `add`, `sub`, `and`, `or`, `sll`, `srl`, `move`. 
 
May be more efficient to use constants than storing values to memory `addi` (add immediate, this means you can incorporate a number without having to get the number from memory) 

All instructions are fixed length 32-bits long. 
> _Note_: Not the case for x86 architectures, which might be variable length

```mips
# What 'a = a + 1' looks like in MIPS

lw  r4, 32(r29)      # The address lives at $sp + 32 bit offset (accessing a local variable), load the value at the address into r4
li  r5, 1            # load immediate a 1 to r5 
add r4, r4, r5       # add r4 and r5, store result in r4 
sw  r4, 32(r29)      # store contents of r4 back address of stack pointer

```

### Registers
32 general purpose registers, including `r0` which is hardwired to return zero, `r31` which the link register for `jal` (where we have to return to for subroutine)

Then there are additional specific registers: 
- `hi/lo` registers for multiplications. (because the issue is a 32 bit mul or div 32 bit may end up with a 64 bit instruciton)
- `pc`/program counter, which is not directly visible (implicitly modified by jumps and branches)


### Branch delay slot/pipelining
> _Note_: Not all ISAs have this. MIPS R3000 is an older architecture which is why it has this, for example x86 doesn't have it but instead does a lot of speculative execution to try to get throughput instead. 

The statement after a branch or jump is also executed.

```mips
li r2, 1
sw r0, (r3)
j 1f           
li r2, 2    # This instruction also executes after the jump, _before_ the jump happens, so 2 is loaded to memory, not 1
li r2, 3

1: sw r2, (r3)
```

The reason for this is that: 
- There is a 5 stage pipeline and processing of instruction is overlapped. So by the time the jump instruction is part way processing, the next instruction is already loaded for executing

- Sometimes you have to insert a `nop` in the assembly to delay the execution. 

> See more in computer architecture, this is just some intuition for why the processor behaves in this apparently counterintuitive way. 

### How jump and link works

```mips
jal 1f 
nop # Sometimes compiler can't think of what to put here so it puts a nop
lw r4, (r6) 

1: 
sw r2, (r3)
jr r31                  
nop 
```
> _Note_: `jal` sets `r31 = pc + 8` (i.e. it sets the return address to 8 bytes from where the `jal` is. This is because the instruction at `pc + 4` is in the branch delay slot so it would be executed already, so we need to the go to the instruction after that)


### Register conventions 
> This is just what the compiler uses, not enforced in the hardware 

- `r2`, `r3`: `v0`, `v1` where return values for a subroutine are put

- `r4` to `r7`: `a0` to `a3` arguments for parameters for a subroutine 

- `r8` to `r15`, `r24`, `r25`:  temporary registers (subroutine can use without saving)

- `r16` to `r23`: `s0` to `s7` subroutine register variables, subroutine must restore them before it exits (subroutine needs to save the values and put them back)

- `r28`: `gp` global pointer, `r29`: `sp` stack pointer, `r30`: `fp` frame pointer
> Last three need ot be restored


### Demo: read some assembler
> TODO: Fix this 
```mips
#  Calculating a factorial in MIPS
blez    a0, 30 <fact+0x30>          # if (n < 0) return;  
addiu   a0, a0, 1                   # n = n + 1; 
li      v1, 1                       # int i = 1;
li      v0, 1                       # int r = 1; 

# Basically this is the for-loop for the factorial
mult    v0, v1                      # r = r * i; 
addiu   v1, v1, 1                   # .label:
                                    #   i = i + 1;
mflo    v0                          # // Put lower 32 bits of mul result to r; 
nop 
bne     v1, a0, 14 <fact+0x14>      # if (i != n) goto label;

mult    v0, v1                      # r = r * i; 
jr      ra                          # return; 
nop
jr      ra                          # return; 
li      v0, 1                       
``` 

### Function calls, stacks, what's in a frame
`fp` points to the current frame on the stack, `sp` points to the top of the stack (which is the end of the current frame) 
> Example: if a function gets called, then `fp` get set to `sp` after a new frame is allocated

Within a stack frame there is usually some convention as to the structure of the frame. 
- arguments $\rightarrow$ saved registers $\rightarrow$ local variables $\rightarrow$ dynamic area (allocate with `alloca`)

> `alloca` can be used to allocate _more_ memory on the current frame (but the issue here is then, sure it gets cleaned up but what you still have a pointer hanging around to this freed memory?)

> _Also note_: your underlying mental model of the stack growing down and the heap growing doesn't exist in practice, this is also an illusion the virtual memory subsystem provides. 