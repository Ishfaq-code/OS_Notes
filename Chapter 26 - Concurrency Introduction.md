State of a Single Thread:
- Program Counter(PC) for fetching and running and instruction
- Private set of registers for own computation

If two threads run on the same processor, there must be context switch between them
- Similar to process context switch
- Registers of T1 saves and registers of T2 restored before running        
- TCB/Thread control block used to store state of each process
	- Address space remains the same
	- No need to switch which page table we are using
	- One stack per thread 
		- Thread local storage

**Why Threads**
- Parallelism
	- Task of transforming single threaded program into a program that does this sort of work on multiple CPUs 
	- Using a thread per CPOU to do work is natural typical way to make programs run faster on modern hardware
- To avoid blocking program progress due to slow I/O
	- Enables overlap of I/O with other activities within a single program
- Threads share address space making it easy to share data among them


**Thread Creation**
- Can be created to be running or a ready/not running state
- Join the threads
	- Ensure both threads run to completion before running main thread again
- Thread created first does not necessarily run first
	- If arbitrary, it is decided by the OS scheduler
- Similar to function call but does not start running immediately, instead it is created as part of the routine and decided to run by the OS scheduler

**Shared Data**
- Shared Data changes on every run due to how OS is scheduling

**Transaction**

**Uncontrolled OS Scheduling**
- Interrupts can mess up how data is handled
	- A data can be incremented in a thread but not applied to actual variable
	- Different thread takes in original number and increments it
	- Incremented twice but went up by once
		- Timer interrupt
		- Virtualized private registers per thread so each thread has a copy of its own variables and locals and such
	- Race Condition - Results depend on the timing of the code's execution 
		- Deterministic computing
		- Indeterminate results
	- Critical Section: Section of code where if multiple threads run it, a race condition may occur
	- Mutual Exclusion Needed - Only 1 thread running CS at a time
	- All terms by Edsger Dijkstra 

**Atomicity**
- More powerful instructions in a single step
	- Not interrupted mid instruction
	- Entire instruction is executed 
	- No in between state
- **Crux: How To Support Synchronization**

**Waiting For Another:**
- Threads need to wait for other threads