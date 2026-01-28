#OS #Process #Quiz_1 #Test_1 #Virtualization

**Limited Directed Execution**: 
- Create entry for process list
- Allocate memory for program
- Load program into memory
- Clear registers
- Execute call main()
- Problems:
	- Program can do anything
	- OS can't stop it

**User Mode**
- To not allow process to run anything
- Code that runs in user mode is restricted
- System call to perform privileged operations

**Kernel Mode**
- OS runs in this mode
- Privileged instruction

Executing System Calls:
- Trap instruction
- Jump to Kernel mode
- When finished return from trap inst

Trap Table at boot time to know which code to run inside OS safely

Two phases to LDE protocol:
- At boot time Kernel initializes a trap table and CPU remembers its location for subsequent use
	- By privileged instruction
- Second phase kernel sets up a few things before using a rft instruction to start execution of process
	- CPU switches to user mode to run process'
	- If another system call, another trap is taken

OS not running when process is running 
- Cooperative Approach: Old, Process running too long so CPU takes
	- In cooperative scheduling system, OS regains control of CPU by waiting for a system call or illegal operation of some kind to take place
- Non Cooperative Approach
	- Timer interrupt - process run for too long so interrupt raised
		- Pre configured by interrupt handler


[[Chapter 7 - Scheduling_Intro]]
