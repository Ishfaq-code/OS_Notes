#OS #Process #Quiz_1 #Test_1 #Virtualization 

**Process**: A running program
- One of the most fundamental abstractions 
- Abstraction provided by the OS of a running program
	- Program is lifeless
		- Sits on disk , bunch of instructions waiting to spring into action
	- Operating system takes these bytes and gets them running, transforming the program into something useful

Crux: How to provide illusion of many CPUs?
- Only few physical CPUs
- How can OS provide illusion of a nearly endless supply of CPUs

Virtualizing CPU to run multiple process
- Running one process and then stopping and running another 
- Illusion that multiple CPUs exist
- **Time Sharing:** User can run as many concurrent process as possible but at the cost of performance
	- Each will run more slowly if CPU must be shared
	- By allowing resource to be used for a little by one entity and then a little while by another, the resource can be shared by many
	- Counter part: Space Sharing
		- Resources divided amongst those who want to use it
	- Employed by all modern OS
	- Mechanism
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
- Often access persistent storage device

Common Design Paradigm to separate high level policies from their low level mechanism
- Mechanisms answer how question to a system: how does an operating system perform a context switch
- Policy provides answer to a which question, which process should the operating system run now
- Separating allows one easily to change policies without having to rethink the mechanism 
	- Form of Modularity 


**Process API:**
- Create: Creating new process
- Destroy: Destroy process forcefully
- Wait: Waiting interface to wait for a process to stop running
- Misc Control: Control for other than killing or waiting for a process
	- Suspending
- Status: Get information about a process

**Process Creation**: 
- Load code and any static data into memory/address space of process
	- Initially reside on disk in exe format or flash based SSDs 
		- Read bytes from disk and place them in memory
	- Early done eagerly - load all at once
	- Now lazily - small piece of code at a time
		- Only data needed during program execution
- Allocate memory to run-time stack
	- For local variables, function parameters, return addresses
	- OS allocates this memory and gives it to the process
- Allocate memory to programs heap
	- For dynamically allocated memory
- OS does other initialization tasks
	- Particularly related to input/output
		- UNIX systems have 3 file descriptions, inputs, outputs and error
- Now ready for execution
	- Jump into the `main()` routine

**Loading:** Takes on disk program and reads it into the address space of the process (in memory, disk --> to memory)

**Process States**:
- Running: Process is running on a process executing instructions
- Ready: Ready to run, OS hasn't run them
	- Ready --> Running = Process being scheduled
	- Running --> Ready = Process being descheduled
- Blocked: Process does some kind of operation that makes it not ready to run until some other event takes place
	- Running to Blocked = I/O Initiated
	- Blocked To Ready = I/O Exit
- Ready -> Blocked = Not possible


**OS DS**:
- Process List: All process that are ready and additional info to track which process is currently running 
	- Also keeping track of blocked process of when I/O initiated vs blocked
- Register Context: Holds contents of register for a stopped process
	- When process stopped, registers saved to this memory location
	- Restoring these registers, OS can resume running the p0rocess
- Initial State: State of process when created
- Final/Zombie State: Exited process but not cleaned up
	- UNIX based systems call it a zombie state


[[Chapter 6 - Limited Direct Execution]]