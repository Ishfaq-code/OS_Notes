#OS #Process #Quiz_1 #Test_1 #Virtualization

**Challenges with Time Sharing Virtualization**:
- Performance - need to implement virtualization without adding excessive overhead to the system
- Control - How to run processes efficiently while retaining control over the CPU
	- Without control a process can run forever or access info that should not be allowed

**Crux:** How to efficiently virtualize the CPU with Control
- OS must virtualize the CPU in an efficient manner while retaining control over the system

**Limited Directed Execution**:  Run the program directly on CPU
- Create entry for process list
- Allocate memory for program
- Load program into memory
- Clear registers
- Execute call `main()`
	- Run Main
	- Execute Return from main
- Free Memory of process
- Remove from Process list
- Problems:
	- Program can do anything
		- Process can perform restricted operation such as issuing an I/O request to a disk
	- OS can't stop it

Procedure Call & System Call
- System Calls are procedure calls but with a hidden trap instruction
	- When doing System calls you are performing a procedure call from the chosen library 
	- Library then uses agreed upon calling convention with the Kernel to put the arguments in well known locations (stack, registers) and put system call number into a well known location as well 
	- Then perform trap instruction
	- Code in library after the trap unpacks returns the values and returns the control to the program that issued the system call
	- Parts of C library that make system calls are hand coded in assembly 
		- Need to carefully follow convention in order to process arguments and return the values correctly and execute hardware specific trap instructions
		- Don't need to write assembly code for trap into OS

**Restricted Operation**:
- CRUX: How to perform restricted operations
	- Must be able to perform I/O and some other restricted operations without giving the process complete control over the System
	- OS and Hardware need to work together
- Letting process have all control means OS loses protection

User process doing privileged operation: System Call
- Pioneered on Atlas
- System calls allow kernel to carefully expose certain key pieces of functionality to user programs
	- Accessing file systems
	- Creating and destroying processes
	- Communicating with other processes
	- Allocating more memory
- Most OS provide few hundred calls
	- Early Unix systems exposed a more concise subset of around twenty calls 

**User Mode**
- While running user mode, process can't issue I/O request
	- Doing so raises an exception and kills process
- To not allow process to run anything
- Code that runs in user mode is restricted
- System call to perform privileged operations

**Kernel Mode**
- OS runs in this mode
- Privileged instruction

Executing System Calls:
- Trap instruction
	- Hardware needs to be careful when executing 
		- Make sure to save enough of the caller's registers in order to be able to return correctly
	- On x86, processor will push the program counter, flags and few other registers onto a per process kernel stack.
		- Return from trap will pop these values
- Jump to Kernel mode
- When finished do return from trap instruction which return to the user program while simultaneously reducing privilege level back to user mode

Trap Table at boot time to know which code to run inside OS safely
- Privileged instruction 
- When machine boots up it does so in privileged mode 
- Tells hardware what code to run when certain exceptional events occur
- OS informs hardware of trap handlers - with some special instruction
	- Once informed it remembers the location of these handlers until machine is next rebooted

- System call number to each system call to specify
	- User code responsible for placing the desired system call number in register or specified location on stack 
	- Serves as protection as user mode not jumping to specific address

LDE Mechanism
- OS Boot: Initialize Trap Table - remember address of syscall handler
- OS Run:
	- Create entry for process list
	- Allocate memory from program
	- Load program into memory
	- Setup user stack with argv
	- Fill kernel stack with reg/PC
	- Return from trap
		- Restore registers from Kernel stack
		- Move to user mode
		- Jump to main
			- Run main
			- Call System Call trap into OS
		- Save registers into kernel stack
		- Move to kernel mode
		- Jump to trap handler
	- Handle Trap
	- Do work of syscall
	- Return from trap
		- Restore register from Kernel stack
		- User mode
		- Jump PC after trap
			- Return from main trap (`exit()`)
	- Free memory of process
	- Remove from process list

OS must detect and reject bad calls

Two phases to LDE protocol:
- At boot time Kernel initializes a trap table and CPU remembers its location for subsequent use
	- By privileged instruction
- Second phase kernel sets up a few things before using a rft instruction to start execution of process
	- CPU switches to user mode to run process'
	- If another system call, another trap is taken

Switching Between Processes
CRUX: How to regain control of the CPU
- How can operating systems regain control of CPU so that it can switch between processes
OS not running when process is running 
- Cooperative Approach: Old, Process running too long so CPU takes
	- Taken by Macintosh, Xerox, Altos systems
	- OS trusts the process of the system to behave reasonably
		- Processes running for too long are assumed to periodically give up the CPU so that the OS can decide to run some other task
		- Yield system call: does nothing except to transfer control to the OS so it can run other processes 
	- In cooperative scheduling system, OS regains control of CPU by waiting for a system call or illegal operation of some kind to take place
	- Issue arises if process doesn't make system call
		- Maybe reboot the system (infinite loop situation)
- Non Cooperative Approach
	- Modern OS halt uncooperative programs
	- CRUX: How to gain control without cooperation
		- Process are not being being cooperative
	- Timer interrupt - process run for too long so interrupt raised
		- Timer device programmed to raise an interrupt every so many milliseconds
			- When interrupt raised, currently running process is halted and pre configured interrupt handler runs
			- OS has regained control of the CPU and can do whatever
				- Stop the process and start a different one
			- OS informs hardware of code at boot
			- OS starts timer at boot
				- Privileged operation
				- Feel safe it will eventually have control

Scheduler makes decision if to whether to continue running the currently running process or switch to a different one
- If decision is made to switch, OS executed a low level piece of code which we refer to as a context switch (mechanism)
	- Save a few register values for currently executing process onto kernel stack
	- Restore a few for the soon to be executing process (from the kernel stack)
	- Ensures after return from trap, system resumes execution of another process then returning to old one
	- To save context of currently running process, OS will execute some low level assembly code to save the general purpose registers, PC and kernel stack pointer of the currently running process and then restore said registers, PC and switch to the kernel stack of the soon to be running process
	- By switching stacks, the kernel enters the call to the switch code in the context of one process and returns the context of another
	- OS executed return from trap instruction, the soon to be executing process becomes the currently running process

- A timer interrupt stops Process A and switches the CPU to kernel mode.
- The CPU saves Process A’s current state onto **A’s kernel stack**.
- The OS saves A’s register state from the kernel stack into **A’s process control block (PCB)**.
- The scheduler selects Process B to run next
- The OS restores Process B’s saved register state from **B’s PCB**.
- The CPU switches to **B’s kernel stack**.
- The system returns from the interrupt, restoring registers from B’s kernel stack.
- The CPU switches back to user mode and Process B resumes execution.



[[Chapter 7 - Scheduling_Intro]]
