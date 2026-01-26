# Implementing processes/threads 


<!--

Things to read up on : 
- minimum knowledge needed of hardware to get things working
- typical implementation strategies
    - kernel vs. user thread discussion 
- how context switching is implemented 
-->

> It's difficult to talk about implementation in general terms so we'll talk about how implementation would happen on MIPS R3000

> TODO: Look at how x86 works instead

## Hardware 
> See notes on MIPS R300. The only really important point is that this architecture branch delay slots (this is just a quirk specific to the hardware being emulated, not generally, e.g. x86 doesn't work like this) 

> I'm not _completely_ sure how this affects implementation, we'll get to that

# Implementing processes


# Implementing threads

# Implementing context switching