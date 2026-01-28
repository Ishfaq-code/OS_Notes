#OS #Process #Quiz_1 #Test_1 #Virtualization 

**Process**: A running program
- Abstraction provided by the OS of a running program

Virtualizing CPU to run multiple process
- Illusion that multiple CPUs exist
- **Time Sharing:** User can run as many concurrent process as possible but at the cost of performance
- Need to have low level machinery and high level intelligence


**Mechanism**: Low level machinery, methods or protocols that implement a needed piece of functionality 

**Context Switch**: Gives OS the ability to stop running one program and start running another on a given CPU

**Policies**: Intelligence  -- Algorithms for making some kind of decision within the OS

**Machine State:** What a program can read or update when it is running
- Memory of a program
	- Instructions lie in memory
	- Data for read and write in the memory
	- Address Space
- Registers
	- Instructions can explicitly read or update registers

**Process API:**
- Create: Creating new process
- Destroy: Destroy process forcefully
- Wait: Waiting interface to wait for a process to stop running
- Misc Control: Control for other than killing or waiting for a process
	- Suspending
- Status: Get information about a process

**Process Creation**: 
- Load code and any static data into memory/address space of process
	- Initially reside on disk in exe format
	- Early done eagerly - load all at once
	- Now lazily - small piece of code at a time
- Allocate memory to run-time stack
- Allocate memory to programs heap

**Process States**:
- Running: Process is running on a process executing instructions
- Ready: Ready to run, OS hasn't run them
	- Read --> Running = Process being scheduled
- Blocked: Process does some kind of operation that makes it not ready to run until some other event takes place
	- Ex I/O request to disk

**OS DS**:
- Process List: All process that are ready and additional info to track which process is currently running 
- Register Context: Holds contents of register for a stopped process
- Initial State: State of process when created
- Final/Zombie State: Exited process but not cleaned up


[[Chapter 6 - Limited Direct Execution]]